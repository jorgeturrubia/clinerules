# Features Planifications

## Feature Structure
Cada feature debe ser completamente autónoma con su propia estructura:

```
src/app/features/planifications/
├── components/
│   ├── plan-card/
│   ├── plan-form/
│   ├── plan-calendar/
│   └── plan-progress/
├── pages/
│   ├── plan-list/
│   ├── plan-detail/
│   ├── plan-create/
│   └── plan-edit/
├── services/
│   ├── planification.service.ts
│   ├── plan-calendar.service.ts
│   └── plan-analytics.service.ts
├── models/
│   ├── planification.model.ts
│   ├── training-plan.model.ts
│   └── plan-template.model.ts
├── guards/
│   └── plan-access.guard.ts
├── adapters/
│   ├── planification.adapter.ts
│   └── calendar.adapter.ts
└── planifications.routes.ts
```

## Separation of Concerns
- **Container Components**: Manejan estado y lógica de negocio
- **Presentation Components**: Solo UI, reciben datos via @Input
- **Services**: Lógica de negocio y comunicación con APIs
- **Adapters**: Transformación de datos entre capas

```typescript
// Container Component Example
@Component({
  selector: 'app-plan-list-container',
  standalone: true,
  template: `
    <app-plan-list 
      [plans]="plans()"
      [loading]="loading()"
      (planSelected)="onPlanSelected($event)"
      (createPlan)="onCreatePlan()">
    </app-plan-list>
  `
})
export class PlanListContainerComponent {
  private planificationService = inject(PlanificationService);
  
  plans = signal<TrainingPlan[]>([]);
  loading = signal(false);

  async ngOnInit() {
    await this.loadPlans();
  }

  private async loadPlans() {
    this.loading.set(true);
    try {
      const plans = await this.planificationService.getPlans();
      this.plans.set(plans);
    } finally {
      this.loading.set(false);
    }
  }

  onPlanSelected(plan: TrainingPlan) {
    this.router.navigate(['/planifications', plan.id]);
  }

  onCreatePlan() {
    this.router.navigate(['/planifications/create']);
  }
}
```

## Lazy Routing Implementation
```typescript
// planifications.routes.ts
export const PLANIFICATION_ROUTES: Route[] = [
  {
    path: '',
    loadComponent: () => import('./pages/plan-list/plan-list-container.component')
      .then(c => c.PlanListContainerComponent)
  },
  {
    path: 'create',
    loadComponent: () => import('./pages/plan-create/plan-create.component')
      .then(c => c.PlanCreateComponent),
    canActivate: [CoachGuard]
  },
  {
    path: ':id',
    loadComponent: () => import('./pages/plan-detail/plan-detail.component')
      .then(c => c.PlanDetailComponent)
  },
  {
    path: ':id/edit',
    loadComponent: () => import('./pages/plan-edit/plan-edit.component')
      .then(c => c.PlanEditComponent),
    canActivate: [PlanOwnerGuard]
  }
];
```

## Services Architecture
```typescript
// Planification Service
@Injectable({
  providedIn: 'root'
})
export class PlanificationService {
  private supabase = inject(SupabaseClient);
  private planAdapter = inject(PlanificationAdapter);

  async getPlans(): Promise<TrainingPlan[]> {
    const { data, error } = await this.supabase
      .from('training_plans')
      .select(`
        *,
        training_sessions(*),
        athletes(name, id),
        coaches(name, id)
      `);

    if (error) throw error;
    return data.map(plan => this.planAdapter.toDomain(plan));
  }

  async createPlan(planData: CreatePlanRequest): Promise<TrainingPlan> {
    const dbPlan = this.planAdapter.toDatabase(planData);
    
    const { data, error } = await this.supabase
      .from('training_plans')
      .insert(dbPlan)
      .select()
      .single();

    if (error) throw error;
    return this.planAdapter.toDomain(data);
  }

  async updatePlan(id: string, updates: Partial<TrainingPlan>): Promise<TrainingPlan> {
    const dbUpdates = this.planAdapter.toDatabase(updates);
    
    const { data, error } = await this.supabase
      .from('training_plans')
      .update(dbUpdates)
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return this.planAdapter.toDomain(data);
  }
}
```

## Domain Models
```typescript
// planification.model.ts
export interface TrainingPlan {
  id: string;
  name: string;
  description: string;
  startDate: Date;
  endDate: Date;
  athleteId: string;
  coachId: string;
  status: PlanStatus;
  sessions: TrainingSession[];
  goals: PlanGoal[];
  createdAt: Date;
  updatedAt: Date;
}

export interface TrainingSession {
  id: string;
  planId: string;
  name: string;
  type: SessionType;
  duration: number; // minutes
  intensity: IntensityLevel;
  scheduledDate: Date;
  completed: boolean;
  notes?: string;
  exercises: Exercise[];
}

export interface PlanGoal {
  id: string;
  description: string;
  targetValue: number;
  currentValue: number;
  unit: string;
  deadline: Date;
  achieved: boolean;
}

export enum PlanStatus {
  DRAFT = 'draft',
  ACTIVE = 'active',
  COMPLETED = 'completed',
  PAUSED = 'paused'
}

export enum SessionType {
  CARDIO = 'cardio',
  STRENGTH = 'strength',
  FLEXIBILITY = 'flexibility',
  RECOVERY = 'recovery',
  MIXED = 'mixed'
}
```

## Adapters Pattern
```typescript
// planification.adapter.ts
@Injectable({
  providedIn: 'root'
})
export class PlanificationAdapter {
  toDomain(dbPlan: any): TrainingPlan {
    return {
      id: dbPlan.id,
      name: dbPlan.name,
      description: dbPlan.description,
      startDate: new Date(dbPlan.start_date),
      endDate: new Date(dbPlan.end_date),
      athleteId: dbPlan.athlete_id,
      coachId: dbPlan.coach_id,
      status: dbPlan.status as PlanStatus,
      sessions: dbPlan.training_sessions?.map(this.sessionToDomain) || [],
      goals: dbPlan.goals?.map(this.goalToDomain) || [],
      createdAt: new Date(dbPlan.created_at),
      updatedAt: new Date(dbPlan.updated_at)
    };
  }

  toDatabase(plan: Partial<TrainingPlan>): any {
    return {
      name: plan.name,
      description: plan.description,
      start_date: plan.startDate?.toISOString(),
      end_date: plan.endDate?.toISOString(),
      athlete_id: plan.athleteId,
      coach_id: plan.coachId,
      status: plan.status
    };
  }

  private sessionToDomain(dbSession: any): TrainingSession {
    return {
      id: dbSession.id,
      planId: dbSession.plan_id,
      name: dbSession.name,
      type: dbSession.type as SessionType,
      duration: dbSession.duration,
      intensity: dbSession.intensity,
      scheduledDate: new Date(dbSession.scheduled_date),
      completed: dbSession.completed,
      notes: dbSession.notes,
      exercises: dbSession.exercises || []
    };
  }
}
```

## Injectable Services Strategy
```typescript
// Servicios inyectables por feature
export const PLANIFICATION_PROVIDERS = [
  PlanificationService,
  PlanCalendarService,
  PlanAnalyticsService,
  PlanificationAdapter,
  CalendarAdapter
];

// En el componente standalone
@Component({
  providers: [PLANIFICATION_PROVIDERS]
})
export class PlanificationFeatureComponent {}
```

## State Management
```typescript
// Plan State Service
@Injectable()
export class PlanStateService {
  private currentPlan = signal<TrainingPlan | null>(null);
  private planSessions = signal<TrainingSession[]>([]);
  private planProgress = signal<PlanProgress | null>(null);

  // Read-only signals
  readonly plan = this.currentPlan.asReadonly();
  readonly sessions = this.planSessions.asReadonly();
  readonly progress = this.planProgress.asReadonly();

  setPlan(plan: TrainingPlan) {
    this.currentPlan.set(plan);
    this.planSessions.set(plan.sessions);
    this.calculateProgress();
  }

  updateSession(sessionId: string, updates: Partial<TrainingSession>) {
    const sessions = this.planSessions();
    const updatedSessions = sessions.map(session => 
      session.id === sessionId ? { ...session, ...updates } : session
    );
    this.planSessions.set(updatedSessions);
    this.calculateProgress();
  }

  private calculateProgress() {
    const sessions = this.planSessions();
    const completed = sessions.filter(s => s.completed).length;
    const total = sessions.length;
    
    this.planProgress.set({
      completedSessions: completed,
      totalSessions: total,
      percentage: total > 0 ? (completed / total) * 100 : 0
    });
  }
}
```
