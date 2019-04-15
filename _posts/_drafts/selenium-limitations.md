---
layout: post
title: "Selenium Sucks For Response Headers"
author: Andrew Kulpa
---

Before starting with the limitations, I first have to say that Selenium is generally fantastic at what it does. Its open source, usable in many languages, supports a number of web drivers, and is platform agnostic. That said, I believe the large amount of limitations I have ran into in the past month to stem purely from their scope: emulating user actions.

This scope would make sense if not for the fact that Selenium serves itself as a testing framework for web applications. At their core, web applications testing tools should be able to interact with a tool as a user would. This is something done very well by Selenium. But, it cannot do much outside of that beyond checking cookies. Checking status codes? Nope. What about viewing HTTP request or response headers? Again, it doesn't support this fundamental feature either.

So how exactly does one get around these limitations? Well, in the multitude of GitHub issue discussions, it seems they just recommend to directly request the content and check the non-user oriented information. Alternatively, one easy fix I've found is turning on performance logging then parsing request and response logs from the underlying web driver. Consider the following C# code snippet:

```c#
var chromeOptions = new ChromeOptions();
ChromePerformanceLoggingPreferences logPrefs = new ChromePerformanceLoggingPreferences();
logPrefs.AddTracingCategories(new string[] { "devtools.timeline" });
ChromeOptions chromeOptions = new ChromeOptions();
chromeOptions.PerformanceLoggingPreferences = logPrefs;
chromeOptions.SetLoggingPreference("performance", LogLevel.All);
chromeOptions.AddArguments("headless");
driver = new ChromeDriver(chromeOptions){
    Url = urlToRequest
};
```

By turning on the Performance Logs in the Chrome web driver, the information is at least available. Still, this is only half of the problem. Each response log needs now to be stored in such a manner to allow for instantaneous lookups of response headers and status codes. I would recommend pulling out the base information needed from the logs and storing it in a dictionary with the URI as the key. A simple class below handles the values:
```C#
public class ResponseData
{
    public string url;
    public int statusCode;
    public dynamic headers;


    public ResponseData(string url, int statusCode, dynamic headers)
    {
        this.url = url;
        this.statusCode = statusCode;
        this.headers = headers;
    }
}
```

Using a JSON deserializer like that found in the `Newtonsoft.Json` package, log entries can now be parsed. Thankfully, logs of specific message method types are fairly standard. For retrieving responses, the message method is of type `Network.responseReceived`. By iterating through each log entry following a request, this can be easily achieved as shown below.

```C#
public Dictionary<Uri, ResponseData> getResponseData(ChromeDriver driver)
{
    Dictionary<Uri, ResponseData> responseData = new Dictionary<Uri, ResponseData> { };
    foreach(LogEntry log in driver.Manage().Logs.GetLog("performance")){
        dynamic logDeserialized = JsonConvert.DeserializeObject(log.Message);
        dynamic message = logDeserialized.message;
        string method = message.method;
        if (method != null && "Network.responseReceived".Equals(method)){
            dynamic parameters = message["params"];
            dynamic response = parameters.response;
            string responseURL = response.url;
            int statusCode = response.status;
            if(!responseData.ContainsKey(new Uri(responseURL))){
                responseData.Add(
                    new Uri(responseURL), 
                    new ResponseData(
                        responseURL, 
                        statusCode, 
                        response.headers
                    )
                );
            }
        }
    }
    return responseData;
}
```

With all that done, the base limitations of Selenium are pretty well circumvented. By generating the dictionary of response data after each request, both status codes and headers can be retrieved easily.