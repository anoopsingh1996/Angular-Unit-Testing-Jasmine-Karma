# Angular-Unit-Testing-Jasmine-Karma

Unit testing in your Angular applications. You will learn about Angular mocks, Karma, and Jasmine and learn how to use them to carry out unit testing and e2e testing.

When you create the project all the dependencies get installed among them everything you are going to need to create the tests.

![1_mSGcvJ7zksygPY3Czn_6QA](https://user-images.githubusercontent.com/9657488/94363130-c05c9f00-00dd-11eb-9bb0-98a5bc7f654a.png)



In the image above you can see all the dependencies installed for testing purposes. Let’s go through the more important ones;

<b>1. jasmine-core:</b> Jasmine is the framework we are going to use to create our tests. It has a bunch of functionalities to allow us the write different kinds of tests.


<b>2. karma:</b>  Karma is a task runner for our tests. It uses a configuration file in order to set the startup file, the reporters, the testing framework, the browser among other things.


<b>3. </b> The rest of the dependencies are mainly reporters for our tests, tools to use karma and jasmine and browser launchers.



To run the test you only need to run the command “ng test”. This command is going to execute the tests, open the browser, show a console and a browser report and, not less important, leave the execution of the test in watch mode.



## Karma Config
Let’s take a look at the karma configuration file created by angular-cli.


![1_cEXorrUPSLypAVtQ_hcUxA](https://user-images.githubusercontent.com/9657488/94363218-6b6d5880-00de-11eb-9139-86d3745e28e3.png)



You probably can guess what most of these configuration properties are for, but let’s go through some of them.

<b>frameworks:</b> this is where jasmine gets set as a testing framework. If you want to use another framework this is the place to do it.

<b>reporters:</b> this is where you set the reporters. You can change them or add new ones.

<b>autoWatch:</b> if this is set to true, the tests run in watch mode. If you change any test and save the file the tests are re-build and re-run.

<b>browsers:</b> this is where you set the browser where the test should run. By default it is chrome but you can install and use other browser launchers.

Let's Start: 

List of methods in our students.service.ts file:

1. getUserList [ get list of users ]

2. getUserDetails [ get User details for id ]


3. transformResponseToAddUniversity [ add “university” property to response]

4. getDepartmentMapping [ get department association by passing “deptId” and “id ]

5.saveUserAssociation [ POST call to save the data ].


```
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root',
})
export class StudentsService {
  constructor(private _http: HttpClient) {}

  getUserList(): Observable<any> {
    return this._http.get('https://reqres.in/api/users');
  }

  getUserDetails(id) {
    return this._http.get(`https://reqres.in/api/users/${id}`)
          .pipe(map((data) => this.transformResponseToAddUniversity(data)));
  }

  // this method transforms the response and adds "university" property to the json
  transformResponseToAddUniversity(response) {
    response['data']['university'] = 'MIT';
    return response;
  }

  // JUST FOR TESTING PURPOSE, NOT USED ANYWHERE IN PROJECT
  getDepartmentMapping(deptId, studentId) {
    let param = new HttpParams();
    param = param.append('deptId', deptId);
    param = param.append('userId', studentId);
    return this._http.get(`https://someUrl.com/association/`, {
      params: param,
    });
  }

  // JUST FOR TESTING PURPOSE, NOT USED ANYWHERE IN PROJECT
  saveUserAssociation(deptId, studentId) {
    return this._http.post(`https://someUrl.com/association/`, {
      deptId: deptId,
      studentId: studentId,
    });
  }
}
```

Let’s start with a basic skeleton of the service:

```
import { TestBed, getTestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { StudentsService } from './students.service';

describe('StudentsService', () => {
  let injector: TestBed;
  let service: StudentsService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [StudentsService],
    });

    injector = getTestBed();
    service = injector.get(StudentsService);
    httpMock = injector.get(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });  
});
```

Note that :

HttpClientTestingModule is imported to mock HttpClientModule because we don’t want to make actual http requests while testing the service.

HttpTestingController is injected into tests, that allows for mocking and flushing of requests.

httpMock.verify() is called after each tests to verify that there are no outstanding http calls.


<b> Testing of getUserList() </b>

```
const dummyUserListResponse = {
  data: [
    { id: 1, first_name: 'George', last_name: 'Bluth', avatar: 'https://s3.amazonaws.com/uifaces/faces/twitter/calebogden/128.jpg' },
    { id: 2, first_name: 'Janet', last_name: 'Weaver', avatar: 'https://s3.amazonaws.com/uifaces/faces/twitter/josephstein/128.jpg' },
    { id: 3, first_name: 'Emma', last_name: 'Wong', avatar: 'https://s3.amazonaws.com/uifaces/faces/twitter/olegpogodaev/128.jpg' },
  ],
};  

it('getUserList() should return data', () => {
    service.getUserList().subscribe((res) => {
      expect(res).toEqual(dummyUserListResponse);
    });

    const req = httpMock.expectOne('https://reqres.in/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(dummyUserListResponse);
  });

```

<b> Testing of getUserDetails() </b>

This method is something special where we are transforming the response of http by another method transformResponseToAddUniversity().

We have the code in students.service.ts as below:

```
  getUserDetails(id) {
    return this._http.get(`https://reqres.in/api/users/${id}`)
          // adding transformation using "pipe"
          .pipe(map((data) => this.transformResponseToAddUniversity(data)));
  }

  // this method transforms the response and adds "university" property to the json
  transformResponseToAddUniversity(response) {
    response['data']['university'] = 'MIT';
    return response;
  }
```

As we can see, we are adding “university” property into the http response. To test this behavior, we need to create 2 response:

1. The response that we expect from http . [dummyUserDetails]

2. The transformed response that we get out of the method [tranformedDummyUserDetails ]


```
const dummyUserDetails = {
  data: { id: 1, 
    first_name: 'George', 
    last_name: 'Bluth', 
    avatar: 'https://s3.amazonaws.com/uifaces/faces/twitter/calebogden/128.jpg' 
  }
};

const tranformedDummyUserDetails = {
  data: {
    id: 1,
    first_name: 'George',
    last_name: 'Bluth',
    avatar: 'https://s3.amazonaws.com/uifaces/faces/twitter/calebogden/128.jpg',
    university: 'MIT',
  },
};
  
  
  it('getUserDetails() should return trasnformed data', () => {
    service.getUserDetails('1').subscribe((res) => {
      // Note that we are expecting "transformed" response with "university" property
      expect(res).toEqual(tranformedDummyUserDetails); 
    });

    const req = httpMock.expectOne('https://reqres.in/api/users/1');
    expect(req.request.method).toBe('GET');
    // Note That we are flushing dummy "http" response
    req.flush(dummyUserDetails); 
  });

```

The getDepartmentMapping() is slightly different, as we are passing params to GET request using HttpParam().


```
  // JUST FOR TESTING PURPOSE, NOT USED ANYWHERE IN PROJECT
  getDepartmentMapping(deptId, studentId) {
    let param = new HttpParams();
    param = param.append('deptId', deptId);
    param = param.append('userId', studentId);
    return this._http.get(`https://someUrl.com/association/`, {
      params: param,
    });
  }
  ```
  
  Note that in getUserDetails() we are simply calling a REST API, but here things are different. We are passing params using HttpParams.
  
  ```
    it('getDepartmentMapping() should return data', () => {
    service.getDepartmentMapping('dept-1', 'usr-1').subscribe((res) => {
      expect(res).toEqual({ id: 'usr-1' });
    });

    const reqMock = httpMock.expectOne((req) => req.method === 'GET' && req.url === 'https://someUrl.com/association/');
    expect(reqMock.request.method).toBe('GET');
    reqMock.flush({ id: 'usr-1' });
  });
  ```
  
<b>  Testing of saveUserAssociation() </b>

```


  it('saveUserAssociation() should POST and return data', () => {
    service.saveUserAssociation('dept-1', 'usr-1').subscribe((res) => {
      expect(res).toEqual({ msg: 'success' });
    });

    const req = httpMock.expectOne('https://someUrl.com/association/');
    expect(req.request.method).toBe('POST');
    req.flush({ msg: 'success' });
  });
  ```
