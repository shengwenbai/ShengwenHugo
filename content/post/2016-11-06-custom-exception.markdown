---

layout:     post

title:      "Create a custom .NET exception"

subtitle:   ""

date:       2016-11-06 12:00:00

author:     "Shengwen"

header-img: "img/home-bg.jpg"

tags:

    - ASP.NET
    - C#
---



1. Create a class that derives from System.Exception class. As a convention, end the class name with Exception suffix. All .NET exceptions end with,  exception suffix. If you don't do so, you won't get a compiler error, but you will be deviating from the guidelines for creating custom exceptions.
```csharp
public class UserAlreadyLoggedInException: Exception{}
```

2. Provide a public constructor, that takes in a string parameter. This constructor simply passes the string parameter, to the base exception class constructor.
```csharp
public UserAlreadyLoggedInException(string message)
    :base(message){}
```

3. Using InnerExceptions, you can also track back the original exception. If you want to provide this capability for your custom exception class, then overload the constructor as shown below. 
```csharp
public UserAlreadyLoggedInException(string message, Exception innerException)
    :base(message, innerException){}
```

4. If you want your Exception class object to work across application domains, then the object must be serializable. To make your exception class serializable mark it with Serializable attribute and provide a constructor that invokes the base Exception class constructor that takes in SerializationInfo and StreamingContext objects as parameters.
```csharp
[Serializable]
public class UserAlreadyLoggedInException: Exception
{
    public UserAlreadyLoggedInException(string message): base(message)
    {
    }

    public UserAlreadyLoggedInException(string message, Exception innerException)
        :base(message, innerException)
    {
    }
    
    public UserAlreadyLoggedInException(SerializationInfo info, StreamingContext context)
        :base(info, context)
    {
    }
}
```

Reference: [kudvenkat's youtube video](https://www.youtube.com/watch?v=9qHb-2Edg7o&list=PLAC325451207E3105&index=42)