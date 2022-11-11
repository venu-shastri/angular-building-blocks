## Writing tests using Jasmine

> Jasmine is a behavior-driven testing framework for browser and Node.js tests. It supports elemental unit testing needs, such as test fixtures, asserts, mocks, spies, and reporters.



###  Unit tests should adhere to the FIRST principle

----

- Fast
- Isolated
- Repeatable
- Self-verifying
- Timely

### Essential Jasmine functions

---

##### describe

You use the describe function to group together a series of tests. This group of tests is known as a test suite. The describe function takes two parameters, a string and a callback function, in the following format:

**describe(string describing the test suite, callback);**

You can have as many describe functions as you want. The number of describe functions depends on how you want to organize your tests into suites. To organize your tests, you also can nest as many describe functions as you want.

**it**
You use the  it function when you want to create a specific test, which usually goes inside a describe function. Like the describe function, the it function takes two parameters— a string and a callback function—using the following format:

**it(string describing the test, callback);**

You create a test inside an it function by putting an assertion inside the callback function. You create an assertion by using the expect function

**expect**
The expect function comes into play in the code that confirms that the test works. These lines of code are also known as the assertion because you’re asserting something as being true. In Jasmine, the assertion is in two parts: the **expect function** and the **matcher**. The expect function is where you pass in the value; for example, a Boolean value true. The matcher function is where you put the expected value. Matcher function examples include **toBe(), toContain(), toThrow(), toEqual(), toBeTruthy(),**
**toBeNull()**



```javascript
describe('testsuite', () => {
    it('First Jasmine test', () => {
    	expect(true).toBe(true);
    });
});



```

##### Fixtures

Every unit test has three parts: arrange, act, and assert . The arrange part of unit tests can be repetitive as multiple test cases often require the same setup. Jasmine provides **fixtures** to help reduce the amount of repetition .

Types Of Fixtures

- **beforeAll**() – runs before all specs in describe
-  **afterAll**() – runs after all specs in describe per test fixture
- **beforeEach**() – runs before each spec in describe
-  **afterEach**() – runs after each spec in describe

The fixtures execute before and after a spec or a group of specs as scoped with their describe block

### Testing Components

---

##### Setup Test Environment

- DebugElement : We can use DebugElement to inspect an element during testing.  DebugElement is like native HTMLElement
  with additional methods and properties that can be useful for debugging elements.
- ComponentFixture :  Used to create a Fixture
- TestBed : set up and configure  tests ,The `TestBed` creates a dynamically-constructed Angular *test* module that emulates an Angular [@NgModule](https://angular.io/guide/ngmodules).
- fakeAsync : ensure that all asynchronous tasks are completed before executing the assertions.
- By : use to select DOM elements. 
- BrowserDynamicTestingModule  : Helps to bootstrap the browser to be used for testing
- RouterTestingModule : set up routing for testing.

```javascript
//Essential Import for Testing 
import { DebugElement } from '@angular/core';
import { ComponentFixture, fakeAsync, TestBed, tick } from '@angular/core/testing';
import { By } from '@angular/platform-browser'; 
import { BrowserDynamicTestingModule } from '@angular/platform-browser-dynamic/testing';
import { RouterTestingModule } from '@angular/router/testing';
```

### Anatomy of auto-generated unit tests

---

- Angular's default configuration leverages **TestBed**
  - Angular-specific component that facilitates the provision of modules, dependency injection, mocking, the triggering of Angular life-cycle events like ngOnInit, and the execution of template logic

ex:-

```javascript
//CurrentWeatherComponent.spec.ts
describe('CurrentWeatherComponent', () => {
    let component: CurrentWeatherComponent
    let fixture: ComponentFixture<CurrentWeatherComponent>
    
    beforeEach(
        async(() => {
            TestBed.configureTestingModule({
            declarations: [CurrentWeatherComponent],
            }).compileComponents()
        })
    )
    
	beforeEach(() => {
            fixture = TestBed.createComponent(CurrentWeatherComponent)
            component = fixture.componentInstance
            fixture.detectChanges()
	})
	it('should create', () => {
			expect(component).toBeTruthy()
	})
})
The first beforeEach function declares and compiles the component's dependent modules asynchronously, while the second beforeEach function creates a test fixture and starts listening to changes in the component,
ready to run the tests once the compilation is complete

//WeatherService.spec.ts

describe('WeatherService', () => {
    let service: WeatherService
    beforeEach(() => {
        TestBed.configureTestingModule({})
        service = TestBed.inject(WeatherService);
    })
    
    it('should be created', () => {
    expect(service).toBeTruthy()
    })
    
    )
})
```

#### Configuring Testbed

> **Testbed** has three major features that assist you in creating unit-testable components:
> • **Declarations** – builds component classes, along with their template logic, to facilitate testing
> • **Providers** – provides component classes without template logic and dependencies that need to be injected
> • **Imports** – imports support modules to be able to render template logic or other platform functionality



ex:-

```javascript
//Declarations
TestBed.configureTestingModule({
declarations: [AppComponent, CurrentWeatherComponent],
}).compileComponents

//Providers
beforeEach(async(() => {
    TestBed.configureTestingModule({
        declarations: [...],
        providers: [WeatherService],
        .......
     
//Imports
import { HttpClientTestingModule } from '@angular/common/http/testing'
...
describe(' CurrentWeatherComponent', () => {
    beforeEach(() => {
    TestBed.configureTestingModule({
    imports: [HttpClientTestingM...
    })
...
    })
    ...


```



#### Test doubles

----

- Fakes
- Mocks, stubs or spies 



##### Fakes

---

A fake is an alternative, simplified implementation of an existing class. It's like a fake service. a fake is instantiated and is used like
the real class

```javascript
export interface ICurrentWeather {
  city: string
  country: string
  date: number
  image: string
  temperature: number
  description: string
}

interface ICurrentWeatherData {
  weather: [
    {
      description: string
      icon: string
    }
  ]
  main: {
    temp: number
  }
  sys: {
    country: string
  }
  dt: number
  name: string
}

export interface IWeatherService {
  getCurrentWeather(city: string, country: string): Observable<ICurrentWeather>
}

//Actual Service 
import { HttpClient, HttpParams } from '@angular/common/http'
import { Injectable } from '@angular/core'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

import { environment } from '../../environments/environment'
import { ICurrentWeather } from '../interfaces'


@Injectable({
  providedIn: 'root',
})
export class WeatherService implements IWeatherService {
  constructor(private httpClient: HttpClient) {}

  getCurrentWeather(city: string, country: string): Observable<ICurrentWeather> {
    const uriParams = new HttpParams()
      .set('q', `${city},${country}`)
      .set('appid', environment.appId)

    return this.httpClient
      .get<ICurrentWeatherData>(
        `${environment.baseUrl}api.openweathermap.org/data/2.5/weather`,
        { params: uriParams }
      )
      .pipe(map((data) => this.transformToICurrentWeather(data)))
  }

  private transformToICurrentWeather(data: ICurrentWeatherData): ICurrentWeather {
    return {
      city: data.name,
      country: data.sys.country,
      date: data.dt * 1000,
      image: `http://openweathermap.org/img/w/${data.weather[0].icon}.png`,
      temperature: this.convertKelvinToFahrenheit(data.main.temp),
      description: data.weather[0].description,
    }
  }

  private convertKelvinToFahrenheit(kelvin: number): number {
    return (kelvin * 9) / 5 - 459.67
  }
}



//Fake Service
import { Observable, of } from 'rxjs'

import { ICurrentWeather } from '../interfaces'
import { IWeatherService } from './weather.service'

export const fakeWeather: ICurrentWeather = {
  city: 'Bethesda',
  country: 'US',
  date: 1485789600,
  image: '',
  temperature: 280.32,
  description: 'light intensity drizzle',
}

export class WeatherServiceFake implements IWeatherService {
  public getCurrentWeather(city: string, country: string): Observable<ICurrentWeather> {
    return of(fakeWeather)
  }
}

//Inject FakeService
src/app/current-weather/current-weather.component.spec.ts
...
beforeEach(
    async(() => {
    TestBed.configureTestingModule({
    ...
    providers: [{
    provide: WeatherService, useClass: WeatherServiceFake
    }],
...

```



##### Mocks,stubs and spies

---

Mocks are configured in the unit test file to respond to specific function calls with a set of responses that can be made to vary from test to test with ease

```JavaScript
import { ComponentFixture, TestBed, waitForAsync } from '@angular/core/testing'
import { By } from '@angular/platform-browser'
import { injectSpy } from 'angular-unit-test-helper'
import { of } from 'rxjs'

import { WeatherService } from '../weather/weather.service'
import { fakeWeather } from '../weather/weather.service.fake'
import { CurrentWeatherComponent } from './current-weather.component'

describe('CurrentWeatherComponent', () => {
  let component: CurrentWeatherComponent
  let fixture: ComponentFixture<CurrentWeatherComponent>
  let weatherServiceMock: jasmine.SpyObj<WeatherService>

  beforeEach(
    waitForAsync(() => {
      const weatherServiceSpy = jasmine.createSpyObj('WeatherService', [
        'getCurrentWeather',
      ])

      TestBed.configureTestingModule({
        declarations: [CurrentWeatherComponent],
        // imports: [HttpClientTestingModule], // If we have to include HttpClientTestingModule then we're not really writing a unit test, because CurrentWeatherComponent shouldn't know about HttpClient
        providers: [{ provide: WeatherService, useValue: weatherServiceSpy }],
      }).compileComponents()

      // weatherServiceMock = TestBed.inject(WeatherService) as any
      weatherServiceMock = injectSpy(WeatherService)
    })
  )

  beforeEach(() => {
    fixture = TestBed.createComponent(CurrentWeatherComponent)
    component = fixture.componentInstance
  })

  it('should create', () => {
    // Arrange
    weatherServiceMock.getCurrentWeather.and.returnValue(of())

    // Act
    fixture.detectChanges() // triggers ngOnInit

    // Assert
    expect(component).toBeTruthy()
  })

  it('should get currentWeather from weatherService', () => {
    // Arrange
    weatherServiceMock.getCurrentWeather.and.returnValue(of())

    // Act
    fixture.detectChanges() // triggers ngOnInit()

    // Assert
    expect(weatherServiceMock.getCurrentWeather).toHaveBeenCalledTimes(1)
  })

  it('should eagerly load currentWeather in Bethesda from weatherService', () => {
    // Arrange
    weatherServiceMock.getCurrentWeather.and.returnValue(of(fakeWeather))

    // Act
    fixture.detectChanges() // triggers ngOnInit()

    // Assert
    expect(component.current).toBeDefined()
    expect(component.current.city).toEqual('Bethesda')
    expect(component.current.temperature).toEqual(280.32)

    // Assert on DOM
    const debugEl = fixture.debugElement
    const titleEl: HTMLElement = debugEl.query(By.css('span')).nativeElement
    expect(titleEl.textContent).toContain('Bethesda')
  })
})

```
