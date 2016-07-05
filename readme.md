## Instructions reordering

| Instructions types | C#  					| x86 					| C# volatile 			|
| ------------------ | --- 					| --- 					| ----------- 			|
| LoadStore          | :heavy_check_mark:   | :x:   				| :x:           		|
| LoadLoad           | :heavy_check_mark:   | :x:   				| :x:           		|
| StoreStore         | :x:   				| :x:					| :x:           		|
| StoreLoad          | :heavy_check_mark:   | :heavy_check_mark:   	| :heavy_check_mark:   	|

## Fencing
### Full fence
```Csharp
public void Method()
{
  ... instruction ...
  ... instruction ...
  ... instruction ...
  Thread.MemoryBarrier(); -----------------
  ... instruction ...
  ... instruction ...
  ... instruction ...
}
```
![full fence](img/full.jpg)
### Half fence : Read fence
Volatile read = acquire fence (half fence)

> :bulb: **ARF = Acquire Read Follow**

```Csharp
private volatile int _value;

public void Method()
{
  ... instruction ...
  ... instruction ...
  ... instruction ...
  if (_value) -----------------
  {
    ...
  }
  ... instruction ...
  ... instruction ...
  ... instruction ...
}
```
![read fence](img/read.jpg)
### Half fence : write fence
Volatile write = release fence (half fence)
```Csharp
private volatile int _value;

public void Method()
{
  ... instruction ...
  ... instruction ...
  ... instruction ...
  _value = 1;  -----------------
  ... instruction ...
  ... instruction ...
  ... instruction ...
}
```
![write fence](img/write.jpg)

## Example
```CSharp
public class X 
{
  private volatile int _continue;
  
  public void Run()
  {
    _continue = 0;
    while (_continue == 0)
    {
      ...
    }
  }
  
  public void Stop()
  {
    _continue = 1;
  }
}
```
In this case if `_continue` is not defined as `volatile` and if `Run()` and `Stop()` are called from different threads and if the compiler wants to reorder some lines or if CPU wants to reorder the instructions we can have something like this :
```CSharp
public class X 
{
  private int _continue;
  
  public void Run()
  {
    _continue = 0;
    if (_continue != 0)
      return;
    
    while (true)
    {
      ...
    }
  }
  
  public void Stop()
  {
    _continue = 1;
  }
}
```
