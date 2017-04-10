---
layout: default
---

## [](#header-2)Write the Uni-corn application in stip.js.

This tutorial will guide you in writing a web application in [stip.js](https://bit.ly/stipjs)

### The application
<p align="center">
    <img src="/fig/uni-corn/home.png" width="500">
</p>
This application can be used by students to manage their "uni-versity" carreer: keeping track of tasks, meetings en classes.
The calendar and charts overview pages show the progress of the student.

For this application we make use of several JavaScript libraries:
.* [later.js](https://bunkat.github.io/later/) to express repeating schedules
.* [highcharts](https://highcharts.com) to display the progress of the student
.* [twitter bootstrap calendar](https://github.com/Serhioromano/bootstrap-calendar) to show the calendar
.* [twitter bootstrap date+time picker](https://eonasdan.github.io/bootstrap-datetimepicker/)

### Tierless JavaScript: slices + annotations
The code is structured in terms of slices: blocks of code, annotated with a name.
These slices contain tierless JavaScript code: plain JS, where functions can be called and defined in a sequential, local
way without the need for explicit constructs for remote communication.
You can define fixed slices: they will end up on the tier you put it on.
If you're not sure where a slice should be, you can let the tool decide: it will make sure that the slice ends up where it should be!

We define two fixed slices in this case: one for the server and one for the client tier.
The server slice contains the code that reads in a file located on the server and declares the replicated data.
The client tier contains all the code that is responsible for rendering the data in the browser.

```javascript
/* @require [fs later]
   @slice setup */
{
    later.date.localTime();

    /* @observable */
    var courses = [];

    /* @observable */
    function Course(title, duration, time) {
    	this.title = title;
    	this.duration = duration;
    	this.time = time;
    }

    function isValidTimeDescr (descr) {
        var sched = later.parse.text(descr);
        // no error => -1
        return sched.error === -1;
    }

    var dataCourses = fs.readFile('data.json');
    var coursesJSON = JSON.parse(dataCourses);
    coursesJSON.forEach(function (json) {
        if (!isValidTimeDescr(json.time))
            throw new Error('Wrong time description in course');
    	var course = new Course(json.title, json.duration, json.time);
    	courses.push(course);
    })
}
```

```javascript
/* @slice browser */
{
    function displaySchedule() {
		var schedule = [];
		courses.forEach(function (course) {
            var nextDate = calculateNext(course.time);
            var prevDate = calculatePrevious(course.time);
            var endNextDate = addMinutes(nextDate, course.duration);
            var endPrevDate = addMinutes(prevDate, course.duration);
            var next = {title: course.title, start: nextDate.getTime(), end: endNextDate.getTime(), class: "event-info"};
            var prev = {title: course.title, start: prevDate.getTime(), end: endPrevDate.getTime(), class: "event-info"};
            schedule.push(next);
            schedule.push(prev);
        })
		var calendar = $("#calendar").calendar({
			tmpl_path : "tmpls/",
			view : "week",
			events_source: schedule
		});
		calendar.view();
	}

	function addMinutes(date, minutes) {
        var ms = date.getTime();
        return new Date(ms + minutes * 60000);
    }

    function calculateNext(timeDescription) {
        var parsed = later.parse.text(timeDescription);
        var s = later.schedule(parsed);
        var next = s.next(1);
        return new Date(next);
    }

    function calculatePrevious(timeDescription) {
        var parsed = later.parse.text(timeDescription);
        var s = later.schedule(parsed);
        var next = s.prev(1);
        return new Date(next);
    }

}
```

```javascript
/* @config:
    browser: client
    setup: server  */
```