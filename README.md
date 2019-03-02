# Calendar
Google calendar task
I have developed this task using HTML,CSS,Java Script.In this task user can view the current date,current month,current year.User can Navigate to the next date,next month,next year.User can also save their events on a particular date.It includes start of event and end time of event.
Google schedule.html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<title>Google Calendar</title>
	<link href="common/samples.css" rel="stylesheet" />	
	<link href="themes/light.css" rel="stylesheet" />	
</head>
<body>
<center><h2>CALENDAR</h2></center>
		<div style="position: absolute; width: 100%; height: 100%;">
			<div id="calendar" style="height: 100%; width: 100%;"></div>
		</div>
	
	<script src="MindFusion.Scheduling.js" type="text/javascript"></script>
	<script src="GoogleSchedule.js" type="text/javascript"></script>
	<script src="TimeForm.js" type="text/javascript"></script>
</body>
</html>
Google schedule.js
var p = MindFusion.Scheduling;


var calendar = new p.Calendar(document.getElementById("calendar"));

calendar.currentView = p.CalendarView.SingleMonth;

calendar.theme = "light";

//add the start time prefix before each subject
calendar.itemSettings.titleFormat = "%s[hh:mm tt] %h";


calendar.render();

calendar.useForms = false;


calendar.itemDoubleClick.addEventListener(handleItemDoubleClick);


calendar.selectionEnd.addEventListener(handleSelectionEnd);


function handleItemDoubleClick(sender, args)
{
	
	var form = new TimeForm(sender, args.item, "edit");
	form.showForm();
}

function handleSelectionEnd(sender, args)
{
	
	var item = new p.Item();
	item.startTime = args.startTime;
	item.endTime = args.endTime;

	
	var form = new TimeForm(sender, item, "new");
	form.showForm();
}
Time forum.js
var p = MindFusion.Scheduling;
var hoursList;

var TimeForm = function (calendar, item, type)
{
	p.BaseForm.call(this, calendar, item);

	this._id = "TimeForm";
	this._type = type;
	this.headerText = "Appointment";
	
}

TimeForm.prototype = Object.create(p.BaseForm.prototype);
TimeForm.prototype.constructor = TimeForm;

TimeForm.prototype.drawContent = function ()
{
	p.BaseForm.prototype.drawContent.call(this);

	var content = this.getContent();	
	
	var row = this.row();
	row.innerHTML = this.localInfo.subjectCaption;
	content.appendChild(row);
	
	// create a text-area for the item subject
	var textArea = this.createTextArea({ id: "subject", initValue: this.item.subject, events: { keydown: this._areaKeyDown} });
	textArea.element.style.width = "200px";
	this.addControl(textArea);

	row = this.row();
	row.appendChild(textArea.element);
	content.appendChild(row);

	// create a drop-down list for start hours
	row = this.row();
	row.innerHTML = "Start Time";
	content.appendChild(row);

	var control = this.createDropDownList({ id: "start_time", items: this.getHourLabels(), initValue: this.getStartTimeIndex(), addEmptyValue: false });
	control.element.style.width = "200px";
	this.addControl(control);

	row = this.row();
	row.appendChild(control.element);
	content.appendChild(row);

	// create a drop-down list for end time
	row = this.row();
	row.innerHTML = "End Time";
	content.appendChild(row);

	var item = this.item;
	control = this.createDropDownList({ id: "end_time", items: hoursList, initValue: this.getEndTimeIndex(),  addEmptyValue: false});
	control.element.style.width = "200px";
	this.addControl(control);

	row = this.row();
	row.style.margin = "0px 0px 30px 0px";
	row.appendChild(control.element);
	content.appendChild(row);
	
	return content;
};

// create an array of objects to fill the hours drop-down
TimeForm.prototype.getHourLabels = function ()
{
	hoursList = [];
	hoursList.push({ value: 0, text: "12:00am" });
	hoursList.push({ value: 1, text: "12:30am" });
	
	let index = 1;
	
	for(var i = 1; i < 12; i++)
	{
		hoursList.push({ value: index+1, text: i.toString() + ":00am" });
	    hoursList.push({ value: index+2, text: i.toString() + ":30am" });
		
		index += 2;
	}
	
	//add the first afternnon hours
	hoursList.push({ value: index + 1, text: "12:00pm" });
	hoursList.push({ value: index + 2, text: "12:30pm" });
	
	index += 2;
	
	for(i = 1; i < 12; i++)
	{
		hoursList.push({ value: index+1, text: i.toString() + ":00pm" });
	    hoursList.push({ value: index+2, text: i.toString() + ":30pm" });
		
		index += 2;
	}	
	
	return hoursList;
}

// get the index of the current item's rank to set the value of the Ranks drop-down
TimeForm.prototype.getStartTimeIndex = function ()
{
	if (this.item != null && this.item.startTime != null)
	{
		
		let index  = this.item.startTime.__getHours() * 2;
		if(this.item.startTime.__getMinutes() > 0)
			index++;
		return index;		
		
	}
	return -1;
}

TimeForm.prototype.getSubject = function()
{
		return this.item.subject;
}

// get the index of the current item's rank to set the value of the Ranks drop-down
TimeForm.prototype.getEndTimeIndex = function ()
{
	if (this.item != null && this.item.endTime != null)
	{
		let hours = this.item.endTime.__getHours();
		let minutes = this.item.endTime.__getMinutes();
		
		let index = hours * 2;
		
		if (minutes > 0)
			index += 1;
		
		return index;
		
	}
	return -1;
}

// override BaseForm's drawButtons method to create form buttons
TimeForm.prototype.drawButtons = function ()
{
	var thisObj = this;

	var btnSave = this.createButton({
		id: "btnSave",
		text: this.localInfo.saveButtonCaption,
		events: { "click": function click(e)
		{
			return thisObj.onSaveButtonClick(e);
		}
		}
	});

	var btnCancel = this.createButton({
		id: "btnCancel",
		text: this.localInfo.cancelButtonCaption,
		events: { click: function click(e)
		{
			return thisObj.onCancelButtonClick(e);
		}
		}
	});

	var buttons = this.row();
	buttons.classList.add("mfp-buttons-row");
	buttons.appendChild(btnSave.element);
	buttons.appendChild(btnCancel.element);

	return buttons;
};

TimeForm.prototype.onSaveButtonClick = function (e)
{
	// update the item with the form data
	 // update the item with the form data
 var startIndex = +this.getControlValue("start_time");
 var startTime = this.item.startTime.date.clone().addHours(startIndex * 0.5);

 var endIndex = +this.getControlValue("end_time");
 var endTime = this.item.endTime.date.clone().addHours(endIndex * 0.5);

 // if end time is specified, decrease it by one day
 if (endIndex != 0 && this.item.endTime.hour == 0)
  endTime.addDays(-1);

 // check for inconsistent start/end time
 if (startTime.valueOf() > endTime.valueOf())
         endTime = startTime.clone().addHours(1);

 // apply changes 
 this.item.subject = this.getControlValue("subject"); 
 this.item.startTime = startTime;
 this.item.endTime = endTime;

 // if a new item is created, add it to the schedule.items collection
 if (this.type === "new")
  this.calendar.schedule.items.add(this.item);

 // close the form
 this.closeForm();

 // repaint the calendar
 this.calendar.repaint(true);
};

TimeForm.prototype.onCancelButtonClick = function (e)
{
	// close the form
	this.closeForm();
};
light.css
.mfp-planner.light .mfp-selection
{
    background-color:#fff7f8;
}/**
* Popup forms styles
*/
.mfp-popup.light
{
    color: #747474;
    box-shadow: 4px 4px 4px rgba(0,0,0,0.35);
    font-family: Verdana, Arial, Sans-Serif;
    font-size: 8pt;
    background: #fff;
    border: solid 1px #d4d4d4;
}
.mfp-popup.light .mfp-popup-header
{
    color: #747474;
    background-color: #fff;
    border-color:#d4d4d4;
}
.mfp-popup.light .mfp-popup-header a.mfp-close-button
{
    opacity: 0.6;
    filter: alpha(opacity=60);
}
.mfp-popup.light .mfp-popup-header a.mfp-close-button:before
{
    content: 'x';
}
.mfp-popup.light .mfp-popup-header a.mfp-close-button:hover
{
    opacity: 1;
    filter: alpha(opacity=100);
}
.mfp-popup.light .mfp-popup-content
{
    margin: 10px 13px;
}
.mfp-popup.light .mfp-popup-title
{
     color: #818284;    
}
.mfp-popup.light a.mfp-button
{
    display: inline-block;
    position: relative;
    width: 92px;
    height: 24px;
    line-height: 24px;
    margin: 0 5px;
    color: #747474;
    background-color: #fff;
    border: solid 1px #d4d4d4;
    text-align: center;
}
.mfp-popup.light a.mfp-button:hover
{
    color: #747474;
    background-color: #fff;
    border: solid 1px #bdf07c;
}
.mfp-popup.light .mfp-popup-content .mfp-text-area, 
.mfp-popup.light .mfp-popup-content .mfp-text-box, 
.mfp-popup.light .mfp-popup-content .mfp-dropDown-list,
.mfp-popup.light .mfp-popup-content .mfp-checkbox-list
{
    border: 1px solid #d4d4d4;
    vertical-align: middle;
    box-sizing: border-box;
    padding: 4px;
    color:#747474;
    font-family: Verdana, Arial, Sans-Serif;
}
.mfp-popup.light .mfp-popup-content .mfp-text-area:hover, 
.mfp-popup.light .mfp-popup-content .mfp-text-box:hover, 
.mfp-popup.light .mfp-popup-content .mfp-dropDown-list:hover, 
.mfp-popup.light .mfp-popup-content .mfp-checkbox-list:hover
{
   border: solid 1px #bdf07c;
}
.mfp-popup.light .mfp-popup-content label
{
    vertical-align: middle;
    margin-left: 3px;
    margin-right: 6px;
}
.mfp-popup.light .mfp-popup-content .mfp-hr-line
{
    border-bottom: inset 1px #d4d4d4;
    margin: 0 0 8px 0;
    padding: 0 0 8px 0;
}
.mfp-popup.light .mfp-popup-content .mfp-checkbox-list
{
    display: inline-block;
    width: 200px;
    height: 100px;
    overflow: auto;
    border: 1px solid #d4d4d4;
    background-color: #fff;
    color: #747474;
    padding: 1px;
    margin-left: 6px;
}
.mfp-popup.light .mfp-popup-content .mfp-date-box .mfp-dropdown-button, 
.mfp-popup.light .mfp-popup-content .mfp-time-box .mfp-dropdown-button
{
    display: inline-block;
    cursor: pointer;
    height: 23px;
    margin: 0 !important;
    text-decoration: none;
    width: 22px;
    vertical-align: middle;
}
.mfp-popup.light .mfp-dtpicker
{
    background-color: #fff;
    border: solid 1px #d4d4d4;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-header td:hover
{
    cursor: pointer;
    color: #747474;
    background-color: #fff;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-content td
{
    padding: 3px;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-content td.mfp-dtp-padding
{
    color: #747474;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-content td.mfp-dtp-today
{
    border: 1px solid #d4d4d4;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-content td.mfp-dtp-selected
{
    border: solid 1px #bdf07c;
    padding: 1px;
    background-color: #fff;
}
.mfp-popup.light .mfp-dtpicker .mfp-dtpicker-content td:hover
{
    background-color: #bdf07c;
    opacity: 0.6;
    filter: alpha(opacity=60);
    color: #404040;
}
/**
* Horizontal list view styles
*/
.mfp-list-view.light,
.mfp-list-view.light .mfp-bg-cell-header,
.mfp-list-view.light .mfp-header-group,
.mfp-list-view.light .mfp-bg-cell:not(:last-child)
{
    border: none;
}
.mfp-list-view.light .mfp-header
{
    color: #747474;
    background-color: #fff;
    border-bottom:1px solid #d4d4d4;
}
.mfp-list-view.light .mfp-header .mfp-button
{
    border:1px solid #d4d4d4;
}
.mfp-list-view.light .mfp-header-group
{
    color: #747474;
    background-color: #fff;
    text-align: center;
    vertical-align: middle;
    width:50px;
}
.mfp-list-view.light .mfp-bg-cell-header
{
    background-color: #fff;
    color: #747474;
}
.mfp-list-view.light .mfp-content .mfp-wrap .mfp-bg-row .mfp-bg-cell
{
     border-right: 1px solid #d4d4d4;
}
.mfp-list-view.light .mfp-content .mfp-wrap .mfp-week.bottom .mfp-bg-row
{
    border-bottom: 1px solid #d4d4d4;
}/**
* Horizontal timetable view styles
*/
.mfp-horizontal-timetable-view.light,
.mfp-horizontal-timetable-view.light .mfp-header,
.mfp-horizontal-timetable-view.light .mfp-header-timeline,
.mfp-horizontal-timetable-view.light .mfp-column
{
    border: none;
}
.mfp-horizontal-timetable-view.light .mfp-header
{
    background-color: #fff;
    color: #747474;
}
.mfp-horizontal-timetable-view.light .mfp-header .mfp-title-table tr.mfp-header-row,
.mfp-horizontal-timetable-view.light .mfp-header .mfp-title-table tr.mfp-group-row
{
    height:40px;
}
.mfp-horizontal-timetable-view.light .mfp-header .mfp-title-table tr.mfp-item-row,
.mfp-horizontal-timetable-view.light .mfp-header .mfp-title-table tr.mfp-empty-row
{
    background-color: #fff;
}
.mfp-horizontal-timetable-view.light .mfp-header-timeline
{
    background-color: #fff;
    color: #747474;
    border-bottom: solid 1px #d4d4d4;
}
.mfp-horizontal-timetable-view.light .mfp-header .mfp-button
{
    border:1px solid #d4d4d4;
}
.mfp-horizontal-timetable-view.light .mfp-header-timeline .mfp-hour,
.mfp-horizontal-timetable-view.light .mfp-header-timeline .mfp-group-time div:not(:first-child),
.mfp-horizontal-timetable-view.light .mfp-header-timeline .mfp-time
{
    border-left: solid 1px #d4d4d4;
}
.mfp-horizontal-timetable-view.light .mfp-content .mfp-column .mfp-cell
{
    border: 1px solid #d4d4d4;
    border-right-style: none;
    border-top-style: none;
}/**
* Item styles
*/
.mfp-planner.light .mfp-item
{
    border:none;
    background-color: #bdf07c;
    color: #404040;
}
.mfp-planner.light .mfp-item-vertical-detail .mfp-time-indicator-wrapper,
.mfp-planner.light .mfp-item-horizontal-detail .mfp-time-indicator-wrapper
{
    background-color: #fff;
    border:1px solid #bdf07c;
}
.mfp-planner.light .mfp-item-vertical-detail .mfp-time-indicator,
.mfp-planner.light .mfp-item-horizontal-detail .mfp-time-indicator
{
    background-color: #bdf07c;
}
.mfp-planner.light .mfp-selected .mfp-item
{
    background-color: #d1ff97;
}
.mfp-planner.light .mfp-item .mfp-resize-start,
.mfp-planner.light .mfp-item .mfp-resize-end
{
    border-color: #fff;
}
.mfp-planner.light .mfp-recurrence .mfp-icons > span::before
{
    content: " ";
    background-image: url(icons.png);
    background-position: -50px -70px;
    margin: 2px 1px;
    height:13px;
    width:15px;
    display: block;	
}
.mfp-planner.light .mfp-exception .mfp-icons > span::before
{
    content: " ";
    background-image: url(icons.png);
    background-position: -70px -70px;
    margin: 2px 1px;
    height:13px;
    width:15px;
    display: block;
}
.mfp-planner.light .mfp-reminder .mfp-icons > span::after
{
    content: " ";
    background-image: url(icons.png);
    background-position: -330px -70px;
    margin: 2px 1px;
    height:13px;
    width:15px;
    display: block;
}/**
* Vertical list view styles
*/
.mfp-vertical-list-view.light, 
.mfp-vertical-list-view.light .mfp-bg-cell-header,
.mfp-vertical-list-view.light .mfp-header,
.mfp-vertical-list-view.light .mfp-bg-cell:not(:last-child)
{
    border: none;
}
.mfp-vertical-list-view.light .mfp-header
{
    color: #747474;
    background-color: #fff;
}
.mfp-vertical-list-view.light .mfp-header .mfp-button
{
    border:1px solid #d4d4d4;
}
.mfp-vertical-list-view.light .mfp-header-group
{
    color: #747474;
    background-color: #fff;
    height:50px;    
}
.mfp-vertical-list-view.light .mfp-bg-cell-header
{
    background-color: #fff;
    color: #747474;
}
.mfp-vertical-list-view.light .mfp-content .mfp-wrap .mfp-bg-table .mfp-bg-cell
{
     border-right: 1px solid #d4d4d4;
}/**
* Month views styles
*/
.mfp-month-view.light,
.mfp-month-range-view.light,
.mfp-month-view.light .mfp-header-weekdays,
.mfp-month-view.light .mfp-header-weeknumbers
{
    border:none;
}
.mfp-month-view.light .mfp-header,
.mfp-month-range-view.light .mfp-header
{
    color: #747474;
	
}
.mfp-month-view.light .mfp-header .mfp-title,
.mfp-month-range-view.light .mfp-header .mfp-title
{
      background-color: #fff;
      padding:10px;
	  /* new */
	  font-size: 18px;
}
.mfp-month-view.light .mfp-header .mfp-button,
.mfp-month-range-view.light .mfp-header .mfp-button
{
    border:1px solid #d4d4d4;
}
.mfp-month-view.light .mfp-header .mfp-header-weekdays
{
     background-color: #fff;
     color: #747474;
	 /* new */
	 font-size: 16px;
}
.mfp-month-view.light .mfp-header-weeknumbers .mfp-weeknumbers
{
    color: #747474;
    background-color: #fff;
    text-align: center;
    border-right: 1px solid #d4d4d4;
}
.mfp-month-view.light .mfp-content .mfp-week
{
    border-top: 1px solid #d4d4d4;
    background-color: #fff;
}
.mfp-month-view.light .mfp-content .mfp-week:last-child
{
    border-bottom: 1px solid #d4d4d4;
}
.mfp-month-view.light .mfp-bg-cell-header
{
    color: #747474;
	/* new */
	font-size: 16px;
	
}
.mfp-month-view.light .mfp-content .mfp-wrap .mfp-bg-row .mfp-bg-cell
{
     border-right: 1px solid #d4d4d4;
}
.mfp-month-view.light .mfp-bg-cell-header
{
    border-color: transparent;
}/**
* Resource view styles
*/
.mfp-resource-view.light
{
    border:none;
}
.mfp-resource-view.light .mfp-header
{
    color: #747474;
    background-color: #fff;
    border-color:#d4d4d4;
}
.mfp-resource-view.light .mfp-header-group-wrap
{
    background-color: #fff; 
}
.mfp-resource-view.light .mfp-header-group
{
    color: #747474;
    border-color:#d4d4d4;
    background-color: #fff;
    text-align: center;
    vertical-align: middle;
}
.mfp-resource-view.light .mfp-header-group-wrap > div
{
    border-bottom:1px solid #d4d4d4;
}
.mfp-resource-view.light .mfp-header .mfp-timeline:not(:last-child)
{
    border-bottom: solid 1px #d4d4d4;
    padding: 4px 0px;
}
.mfp-resource-view.light .mfp-header .mfp-timeline div
{
    border-right: solid 1px #d4d4d4;
}
.mfp-resource-view.light .mfp-header .mfp-timeline:last-child div
{
    border-right: solid 1px #d4d4d4;
    box-sizing: initial;
}
.mfp-resource-view.light .mfp-header .mfp-header-timeline .mfp-timeline:last-child div
{
    text-align: center;
}
.mfp-resource-view.light .mfp-wrapper .mfp-content .mfp-wrap .mfp-lane-table td
{
	border-bottom: solid 1px #d4d4d4;
	border-right: solid 1px #d4d4d4;
}
.mfp-resource-view.light .mfp-wrapper .mfp-content .mfp-wrap .mfp-lane-table tr:last-child td
{
	border-bottom: solid 1px #d4d4d4;
}
/**
* Vertical timetable styles
*/
.mfp-timetable-view.light,
.mfp-timetable-view.light .mfp-header,
.mfp-timetable-view.light .mfp-header-timeline,
.mfp-timetable-view.light .mfp-column
{
    border: none;
}
.mfp-timetable-view.light .mfp-header
{
    background-color: #fff;
    color: #747474;
}
.mfp-timetable-view.light .mfp-header .mfp-title-table tr.mfp-header-row,
.mfp-timetable-view.light .mfp-header .mfp-title-table tr.mfp-group-row
{
    height:40px;
}
.mfp-timetable-view.light .mfp-header .mfp-title-table tr.mfp-item-row,
.mfp-timetable-view.light .mfp-header .mfp-title-table tr.mfp-empty-row
{
    background-color: #fff;
}
.mfp-timetable-view.light .mfp-header-timeline
{
    background-color: #fff;
    color: #747474;
}
.mfp-timetable-view.light .mfp-header .mfp-button
{
    border:1px solid #d4d4d4;
}
.mfp-timetable-view.light .mfp-header-timeline .mfp-hour,
.mfp-timetable-view.light .mfp-header-timeline .mfp-group-time div:not(:first-child),
.mfp-timetable-view.light .mfp-header-timeline .mfp-time
{
    border-top: solid 1px #d4d4d4;
}
.mfp-timetable-view.light .mfp-content .mfp-column .mfp-cell
{
    border: 1px solid #d4d4d4;
    border-right-style: none;
    border-bottom-style: none;
}/**
* Week view styles
*/
.mfp-week-view.light,
.mfp-week-view.light .mfp-header-weekdays,
.mfp-week-view.light .mfp-bg-cell-header
{
    border:none;
}

.mfp-week-view.light .mfp-header
{
    color: #747474;
}
.mfp-week-view.light .mfp-header .mfp-title
{
    background-color: #fff;
}
.mfp-week-view.light .mfp-header .mfp-title .mfp-link
{
    padding:10px;
}
.mfp-week-view.light .mfp-header .mfp-header-weekdays
{
     background-color: #fff;
     color: #747474;
}
.mfp-week-view.light .mfp-content .mfp-week
{
    border-top: 1px solid #d4d4d4;
    background-color: #fff;
}
.mfp-week-view.light .mfp-bg-cell-header
{
    background-color: #fff;
    color: #747474;
}
.mfp-week-view.light .mfp-content .mfp-wrap .mfp-bg-row .mfp-bg-cell
{
     border-right: 1px solid #d4d4d4;
}
.mfp-week-view.light .mfp-content .mfp-wrap .mfp-week.bottom .mfp-bg-row
{
    border-bottom: 1px solid #d4d4d4;
}


