Call web worker as simple typed class methods. And make DOM events available for worker.

**Requirements:**

```npm install worker-typescript-connector --save```

and worker-loader to start worker:

```npm install worker-loader --save-dev```

**How to use:**

For example we have `ExampleClass` - this is the Class that we want to run inside worker, and it implements example `ExampleInterface`

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
    return `some result string ${someArgument}`;
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
 import { startWorkerInterfaceFor } from 'worker-typescript-connector/dist/src/lib/worker-base';
 import { ExampleClass } from './example-class';
 
 const exampleClass = new ExampleClass();
 
 startWorkerInterfaceFor(exampleClass);
```


**Just it**, now, when call `ExampleWorkerClient` methods it will be called inside worker and return results back:


```
...

async function test() {
    const exampleWorkerClient = new ExampleWorkerClient();
    
    exampleWorkerClient.myMethodOne(1);

    exampleWorkerClient.myMethodTwo('string...', true);

    console.log(await exampleWorkerClient.myMethodWithSharedData('test', canvas)); // Will print "some result string test"
}

...

```
