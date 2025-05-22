# Subscribe Notify pattern

## Subscribe Notify pattern

In Angular apps, we need to subscribe to Observables.

eg. if we are making calls to get/post/put/delete data using **HttpClient**.

This pattern encapsulates the subscribe to the Observable into streams.

A data$ stream & an error$ stream.

So, in your component you only deal with streams.

## Notification Service

![NotificationService](NotificationService.png)

I have a provided a `NotificationService` that implements this pattern.

```TS
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({ providedIn: null })
export class NotificationService<T> {

    public data$: BehaviorSubject<T> = new BehaviorSubject<T>(<T>{});
    public error$: BehaviorSubject<any> = new BehaviorSubject<any>(null);

    public subscribe(observer: Observable<T>) {
        this.resetError();
        observer.subscribe({
            next: (data) => {
                console.log('Data loaded successfully', data);
                this.data$.next(data);
            },
            error: (e) => {
                console.error('Error loading data', e.message);
                this.error$.next(e);
            }
        });
    }

    public resetData() {
        this.data$.next(<T>{});
    }
    public resetError() {
        this.error$.next(null);
    }
    public reset() {
        this.data$.next(<T>{});
        this.error$.next(null);
    }
}
```

## Usage of NotificationService

In your TypeScript component, you inject the NotificationService,

and set local variables to point to it's data$ & error$ streams.

The mark up uses these variables to render.

Then, when you call it's `subscribe` with your Observable,

data & error get published to the streams.

And, the local variables are updated which in turn re-renders the markup.

**component.ts**

```TS
private readonly notificationService = inject(NotificationService<Employee[]>);

private readonly employeeApiService = inject(EmployeeApiService);

public employees$ = this.notificationService.data$;
public error$ = this.notificationService.error$;

getEmployeesByName(searchName: string) {
     // Fetch employees by name.
     this.notificationService.subscribe
     (
        this.employeeApiService.getEmployeesByName(searchName)
     );
}
```

**component.html**

```html
@if (error$ | async) {
   <span style="color:red">(error$ | async)?.message</nz-alert>
}

@for (employee of employees$ | async; track employee.id) {
    <tr>
        <td>{{ employee.name }}</td>
        <td>{{ employee.totalLeaveDays }}</td>
    </tr>
}
```
