---
layout: post
title: "Parsing JSON with Golang"
author: Andrew Kulpa
excerpt: Parsing common data formats like JSON is a very common and necessary task for anyone using a typical RESTful API. In this blog post, we will go through the basics of using JSON with Go.
modified_date: 2020-04-25
# categories: [Go, JSON]
tags: [Go, JSON]
---

Objectives:
  * [Parsing JSON](#parsing-json)
  * [Creating JSON from Go Structs](#creating-json-from-go-structs)
  * [JSON Tags](#json-tags)
  * [Reading JSON Files](#reading-json-files)
  * [Encoding Unstructured Data](#encoding-unstructured-data)

JavaScript Object Notation (JSON) is a very common file format that is both human-readable and easily parsed. I mean, its so likely that you will come across it (if you haven't already) that I wrote this article!

Parsing common data formats like JSON is a very standard and necessary task for anyone using a typical RESTful API. In this blog post, we will go through the basics of using JSON with Go. To do this, we will look at the main JSON data types and how to work with them simply and idiomatically.


## Parsing JSON

The Go standard library provides us with all we need through the json package. When working with any JSON string, the easiest way to parse it is:

```go
import (
	"encoding/json"
	"fmt"
)
jsonString = := `{"key":"val"}`

// &result is the memory address for the variable we will the parsed data
json.Unmarshall([]byte(jsonString), &result)
```

Now typically we would want to formally define the JSON result as some structured data. This is incredibly easy using Go `struct`. Lets say, for instance, we are receiving a json response for a course from a learning management system. Every course has a `courseNumber`, `department`, and `description` field:

```json
{
    "courseNumber": 101,
    "department": "CS",
    "description": "Intro to Computer Science"
}
```

In order for us to work effectively with this data, we need to create a `struct` that conforms to this json schema. With that in mind, we will create a course struct with the same attributes:
```go
type Course struct {
    CourseNumber int
    Department string
    Description string
}
```

Then we will unmarshall just like before:
```go
courseJSONData := []byte(`
    {
        "courseNumber":101,
        "department": "CS",
        "description": "Intro to Computer Science"
    }
`)
var course Course
json.Unmarshal(courseJSONData, &course)
fmt.Printf("CourseNumber: %d, Department: %s, Description: %s\n", course.CourseNumber, course.Department, course.Description)
// prints: CourseNumber: 101, Department: CS, Description: Intro to Computer Science
```

## Creating JSON from Go Structs

Let's say we needed to process the JSON we received in Go and return as JSON data afterwards. This wouldn't be all to uncommon. Well, thankfully creating nearly identical JSON data from the same Go `struct` is even easier! Take for example the following code:

```go
data, _ := json.Marshal(course)
fmt.Print(string(data) + "\n")
// prints: {"CourseNumber":101,"Department":"CS","Description":"Intro to Computer Science"}
```

## JSON Tags
While it is incredibly easy to output JSON from Go `structs`, sometimes you need a little bit more than just spitting out the same exact object. This could be handled by updating the struct in-place or modifying upon inserting into a new struct. The following example demonstrates the latter:
```go

// Course JSON struct meant to alter unmarshal'd JSON data after marshal'ing again
type AlteredCourse struct {
	CourseNumber int    `json:"courseNum"`
	Department   string `json:"dept"`
	Description  string `json:"title"`
}

var alteredCourse = &AlteredCourse{
  CourseNumber: course.CourseNumber,
  Department:   course.Department,
  Description:  course.Description,
}
alteredData, _ := json.Marshal(alteredCourse)
fmt.Print(string(alteredData) + "\n")
// prints: {"courseNum":101,"dept":"CS","Title":"Intro to Computer Science"}
```
As demonstrated above, we first defined a new struct called `AlteredCourse` with something new called JSON tags. These JSON tags allow us to have different JSON object keys than the name of the struct variable that they came from. For instance, instead of outputing the key `CourseNumber` we output `courseNum`. This follows for the remaining JSON tags.


## Reading JSON Files 

## Encoding Unstructured Data

Now the rule of thumb is if you can use a `struct` you should. I won't try and beat that horse any more than the community already does. Instead, we can circumvent the need for a `struct` by using a map. Take a look at the following code snippet:

```go
unstructuredCourseData := map[string]interface{}{
	"courseNumber": 101,
	"department":   "CS",
	"description":  "Intro to Computer Science",
	"enrollees": []map[string]int{
		{
			"id":   555432432,
			"role": 6,
		},
		{
			"id":   555412345,
			"role": 5,
		},
	},
	"books": map[string]interface{}{
		"Intro to Java": map[string]interface{}{
			"description": "Introductory concepts of Programming using Java",
			"required":    false,
			"ISBN":        12345421,
		},
	},
}
unstructuredData, _ := json.Marshal(unstructuredCourseData)
fmt.Print(string(unstructuredData) + "\n")
// prints: {"books":{"Intro to Java":{"ISBN":12345421,"description":"Introductory concepts of Programming using Java","required":false}},"courseNumber":101,"department":"CS","description":"Intro to Computer Science","enrollees":[{"id":555432432,"role":6},{"id":555412345,"role":5}]}
```

In the example above, we create JSON from a rather complex map. While a struct may have made this much more manageable, the difficulty clearly presented itself in the map definition -- not the `Marshal` process. So, if this method ends up being easier, then go for it!


