Call worker as simple typed class methods. And make DOM events available from worker.

How to use:

```npm install worker-typescript-connector --save```


For example we have `ExampleClass` - this is the Class that we want to run inside worker, and it implements our example `ExampleInterface`

example-class.ts:
```
export class ExampleClass implements ExampleInterface {
  
  myMethodOne(someArgument: number): void {
    ...some code here
  }
  myMethodTwo(someArgument: string, anotherArgument: boolean): void {
     ...some sode here
  }
  
  async myMethodWithSharedData(someArgument: string, offscreenCanvas: OffscreenCanvas): Promise<string> {
    return "some result string";
  }
}
```

example-interface.ts:
```
export interface ExampleInterface {
  myMethodOne(someArgument: number): void;
  myMethodTwo(someArgument: string, anotherArgument: boolean): void;

  myMethodWithSharedData(someArgument: string, offscreenCanvas: OffscreenCanvas): Promise<string>;
}
```

Now create worker client - with just method declarations (with empty implementation)

client.ts:
```
import Worker from 'worker-loader!./worker';
import { WorkerClientBase, AutowiredMethodWithSharedData, AutowiredMethod } from 'worker-typescript-connector';
import { ExampleInterface } from '../example-interface';

export class ExampleWorkerClient extends WorkerClientBase implements ExampleInterface {
  constructor() {
    super(new Worker());
  }

  /* 
   *  Call to next methods will be translated to worker calls with all arguments
   *  Implementation do not needed
   */
  @AutowiredMethod
  myMethodOne(someArgument: number): void {}
  @AutowiredMethod /* Call to this method will be translated to worker with all arguments */
  myMethodTwo(someArgument: string, anotherArgument: boolean): void {} /* Implementation do not needed */
  
  /* Call to this method will be translated to worker with all arguments and shared OffscreenCanvas property */
  @AutowiredMethodWithSharedData
  myMethodWithSharedData(someArgument: string, offscreenCanvas: OffscreenCanvas): Promise<string> {
    return null;
  }

```
and worker.ts:
```import { startWorkerInterfaceFor } from 'worker-typescript-connector';
 import { MyClass } from '../my-class';
 
 const myClass = new MyClass();
 
 startWorkerInterfaceFor(myClass);
```

Just it, now, when you will call `ExampleWorkerClient` methods it will be called inside workers and return results back.
