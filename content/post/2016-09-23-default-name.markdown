---
layout:     post
title:      "Set a default name for files that will be downloaded by users"
subtitle:   ""
date:       2016-08-23 12:00:00
author:     "Shengwen"
header-img: "img/contact-bg.jpg"
tags:
    - C#
    - ASP.NET
---

Sometimes we need to generate files dynamicly and let the users ro download these files when running the web app. The users may want our app to generate a default file name to make it easy for them to save the files. We need to use the Content-Diposition Header to achieve that. 
```csharp
public void ProcessRequest(HttpContext context)
{
    byte[] fileContent = GetFileContent();
    context.Response.ContentType = "application/octet-stream";
    string downloadName = "Medical-Evaluation-Result.xlsx";
    string headerValue = string.Format("attachment; filename=\"{0}\"", downloadName);
    context.Response.AddHeader("Content-Disposition", headerValue);
    context.Response.OutputStream.Write(fileContent, 0, fileContent.Length);
}
```

