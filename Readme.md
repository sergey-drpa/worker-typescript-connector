Call worker as simple typed class methods. And make DOM events available from worker.

How to use:

Requirements:

```npm install worker-typescript-connector --save```

and worker-loader to start worker:

```npm install worker-loader --save-dev```



For example we have `ExampleClass` - this is the Class that we want to run inside worker, and it implements our example `ExampleInterface`

example-class.ts:
```
import { ExampleInterface } from './example-interface.ts';

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

Now create worker client - with just method declarations (empty implementation)

client.ts:
```
import Worker from 'worker-loader!./worker';
import { WorkerClientBase, AutowiredMethodWithSharedData, AutowiredMethod } from 'worker-typescript-connector';
import { ExampleInterface } from './example-interface';

export class ExampleWorkerClient extends WorkerClientBase implements ExampleInterface {
  constructor() {
    super(new Worker());
  }

  /* 
   *  Call next methods annotated with @AutowiredMethod will be translated to worker calls with all arguments
   *  Implementation do not needed
   *  
   *  Call methods annotated with @AutowiredMethodWithSharedData will also pass shared data as OffscreenCanvas/ArrayBuffers and etc to worker
   */

  @AutowiredMethod
  myMethodOne(): void {}
  
  @AutowiredMethod
  myMethodTwo(): void {}
 
  @AutowiredMethodWithSharedData
  myMethodWithSharedData(): Promise<string> { return null; }

```
and worker.ts:
```
 import { startWorkerInterfaceFor } from 'worker-typescript-connector';
 import { ExampleClass } from './example-class';
 
 const exampleClass = new ExampleClass();
 
 startWorkerInterfaceFor(exampleClass);
```

Just it, now, when you will call `ExampleWorkerClient` methods it will be called inside worker and return results back.
