# Live Project
## Introduction
For the last six weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC Web Application in C# for three sprints. Working on this project was a great learning oppertunity for fixing bugs, cleaning up code, and adding requested features. I saw how a good developer works with what they have to make a quality product. For the first sprint I worked on several stories that mix of front end, back end and full stack stories. The second sprint I focused on just front end stories with the final sprint just back end. Because much of the site had already been built, there were also a good deal of stories that needed to be completed, all of varying degrees of difficulty. 

Below are descriptions of the stories I worked on, along with code snippets and navigation links. 

## Back End Stories
* [MyJobsButton](#myjobsbutton)
* [ShiftTimeEditing](#shifttimeediting)
* [SessionTimeout](#sessiontimeout)
## MyJobsButton
This story I created a search filter that would show current users jobs only when clicking on the button. There was a button already for showing all jobs.

	View

		<div class="col-3" align="left">
			@*Button that shows current user page view*@		
			@using (Html.BeginForm("Index", "Jobs", FormMethod.Get))
			{
				<p>
					@Html.Hidden("SearchString", "MyJobs")
					<input type="submit" value="MyJobs" />
				</p>
			}
		</div>
			
	Controller

		if (searchString == "MyJobs")
		{
			if (userId != null)
			{
				jobs = db.Jobs.Include(jd => jd.Details).Where(j => j.Manager.Id == userId);
				schedule = db.Schedules.Include(s => s.Job.Details).Where(s => s.Person.Id == userId).Where(s => s.Job.Active == true);
		   
			}
		}

## ShiftTimeEditing
This story I added function to the Edit method to allow the ShiftTimes database to be updated. I added the Bind for the ShiftTime Model and the if statement to update database.
	
			
	Controller	
	
		[HttpPost]
		[ValidateAntiForgeryToken]
		public ActionResult Edit([Bind(Include = "JobIb,JobTitle,JobType,Active,Location,Manager,Details")] Job modelJob,
			[Bind(Include = "ShiftTimeId,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday,Default")] ShiftTime modelShiftTime)
		{
			modelShiftTime.ShiftTimeId = modelJob.JobIb;
			Job job = db.Jobs.Find(modelJob.JobIb);

			JobOther jobOther = db.JobOthers.Find(modelJob.Details.JobOtherId);

			ShiftTime shiftTime = db.ShiftTimes.Find(modelShiftTime.ShiftTimeId);


			if (jobOther == null)
			{
				jobOther = new JobOther();
				jobOther.Job = job;
				jobOther.Note = modelJob.Details.Note;
				db.JobOthers.Add(jobOther);

				db.Entry(job.Details).State = EntityState.Added;
				db.SaveChanges();
			}
			else
			{
				job.Details.Note = modelJob.Details.Note;
				db.Entry(job.Details).State = EntityState.Modified;
				db.SaveChanges();
			}
			
			if (shiftTime == null)
			{
				shiftTime = new ShiftTime();
				shiftTime.ShiftTimeId = job.JobIb;
				db.ShiftTimes.Add(shiftTime);

				db.Entry(shiftTime).State = EntityState.Added;
				db.SaveChanges();
			}
			else            
			{
				shiftTime.Default = modelShiftTime.Default;
				shiftTime.Monday = modelShiftTime.Monday;
				shiftTime.Tuesday = modelShiftTime.Tuesday;
				shiftTime.Wednesday = modelShiftTime.Wednesday;
				shiftTime.Thursday = modelShiftTime.Thursday;
				shiftTime.Friday = modelShiftTime.Friday;
				shiftTime.Saturday = modelShiftTime.Saturday;
				shiftTime.Sunday = modelShiftTime.Sunday;
				db.Entry(shiftTime).State = EntityState.Modified;
				db.SaveChanges();
			}
				
	Edit.cshtml
	
		Here I changed the new {} that had blank modal to populate the modal with data.
		
		<div class="form-group">
				@Html.LabelFor(model => model.WeeklyShifts, htmlAttributes: new { @class = "control-label col-md-2" })
				<div class="col-md-10">
					@Html.TextBoxFor(model => model.WeeklyShifts.Default, new { htmlAttributes = new { @class = "form-control" } })
					@Html.Partial("_shiftTimeModal", Model.WeeklyShifts) //populated the modal with data
					<button type="button" class="btn-base btn-w-100px" data-toggle="modal" data-target="#ShiftTimeModal">Edit</button>
				</div>
			</div>

## SessionTimeout
This story was to implement a timer to log out user after 15 minutes with a 5 minute timer. Also a logout in the profile dropdown.
	Timeout Popup
	
		<div id="divPopupTimeOut">
			<div id="timeOutText" class="row" style="margin-top:10px; margin-left:10px;">
				<h3><span id="CountDownHolder"></span></h3>
				Your session is about to expire!
				<br />
				Click OK to continue your session.
			</div>
			<div class="buttonRow">
				<div class="text-center button-block" style="margin-top:22px;">
					<button type="button" class="btn btn-default btn-sm sessionTimeoutBtn" onclick="SessionTimeout.sendKeepAlive();">OK</button>
					<button type="button" class="btn btn-default btn-sm sessionTimeoutBtn" onclick="SessionTimeout.hidePopup(); javascript:document.getElementById('logoutForm').submit()">Cancel</button>
				</div>
			</div>
		</div>
		
	Script for popup
	
		<script type="text/javascript">
			var loginUrl = '@Url.Action("Login", "Account")';
			var extendMethodUrl='@Url.Action("ExtendSession","Dashboard")';
			$(document).ready(function(){
					SessionTimeout.schedulePopup();
				});

			window.SessionTimeout = (function () {
				var _timeLeft, _popupTimer, _countDownTimer;
				var stopTimers = function() {
					window.clearTimeout(_popupTimer);
					window.clearTimeout(_countDownTimer);
				};
				var updateCountDown = function() {
					var min = Math.floor(_timeLeft / 60);
					var sec = _timeLeft % 60;
					if(sec < 10)
						sec = "0" + sec;

					document.getElementById("CountDownHolder").innerHTML = min + ":" + sec;

					if(_timeLeft > 0) {
						_timeLeft--;
						_countDownTimer = window.setTimeout(updateCountDown, 1000);
					}
					else {
						document.location = loginUrl;
						document.getElementById('logoutForm').submit();
					}
				};
				var showPopup = function() {
				$("#divPopupTimeOut").show();
					_timeLeft = 310;
					updateCountDown();
				};
				var schedulePopup = function() {
				$("#divPopupTimeOut").hide();
					stopTimers();
					_popupTimer = window.setTimeout(showPopup, @PopupShowDelay);
				};
				var hidePopup=function(){
				$("#divPopupTimeOut").hide();
				};
				var sendKeepAlive = function() {
					stopTimers();
					$("#divPopupTimeOut").hide();
					$.ajax({
						type: "GET",
						url: extendMethodUrl,
						contentType: "application/json; charset=utf-8",
						dataType: "json",
						success: function successFunc(response) {
							SessionTimeout.schedulePopup();
						},
						error:function(){
						}
					});
				};
				return {
					schedulePopup: schedulePopup,
					sendKeepAlive: sendKeepAlive,
					hidePopup:hidePopup,
					stopTimers:stopTimers,
				};

			})();

		</script>
		
	Script for logging out user from profile
	
		@using (Html.BeginForm("LogOff", "Account", FormMethod.Post, new { id = "logoutForm" }))
			{
				@Html.AntiForgeryToken()
				<li>
					<a class="dropdown-links" href="javascript:document.getElementById('logoutForm').submit()">Log off</a>
				</li>
			}	
			
	CSS 

		#divPopupTimeOut {
			display: none;
			text-align: center;
			width: 280px !important;
			position: fixed;
			border-radius: 10px;
			top: 0px;
			left: 40%;
			z-index: 9999;
			height: 165px;
			background-color: #EAA6CD;
			box-shadow: 0px 0px 16px 2px rgba(240,255,241,0.67);
			padding:10px;
		}
		.sessionTimeoutBtn {
			cursor: pointer;
			padding: 0px;
			height: 30px;
			width: 100px;
			text-align: center;
			background-color: #A091D4;
			margin-top: -20px;
		}
		buttonRow{
			display:inline-block;
			text-align:center;


*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Page Top](#live-project)*

## Front End Stories
* [NavbarTweak](#navbartweak)
* [ButtonIconTweak](#buttonicontweak)
* [CreatePageButton](#createpagebutton)
* [PersonalProfiles](#personalprofiles)
* [LogoProfileStyling](#logoprofilestyling)
## NavbarTweak
Changed layout to conform with other icons and added tool tip to print icon
		
		New			
			<li>
				<a href="#" title="Print" onclick="window.print()"> <i class="fa fa-print"></i><text class="nav-title">Print</text></a>
			</li>
		
		Old
			<li>
				<div onclick="window.print()">
					<i class="fa fa-print" aria-hidden="true" href="#"></i><text class="nav-sub-title" title="nav-title">Print</text>
				</div>
			</li>
			
## ButtonIconTweak
Three icons, Navbar expand, details button, and several lists all the same fa-list. Changed Navbar and details to be more related to item.
	Navbar expand
	
		View Layout
			<div id="open-menu">
				<i class="fa fa-forward"></i> 
			</div>
			
	Details button				
	
		View Anchor Button
			case ButtonType.Details:
				<a type="button" class="btn btn-sm buttonbackground" href="@Url.Action(Model.Action, new { id = Model.RouteId })">
					<span><i class="fa fa-info-circle"></i></span> //changed from list to info-circle
				</a>
				break;
				
## CreatePageButton
All the Create buttons on the Index pages were outside the container or not displaying correctly. I updated each with this code changing the id="" to the correct create page.

	New Create format
		<div class="form-group">
			<div class="col-md-offset-2 col-md-10">
				<input type="submit" id="addShiftTimes" value="Create" class="btn-base btn-w-100px" />
				@Html.Partial(AnchorButtonGroupHelper.PartialView, AnchorButtonGroupHelper.GetBack())
			</div>
		</div>
	
## PersonalProfiles
There was an index for personal profiles but no way to access them. I added a link for admin to the entire list and one for the current user in the profile dropdown. And then display the data in card format.
	Link to index for admin
	
		<li><a href="@Url.Action("Index", "PersonalProfiles")" title="All Personal Profiles"><i class="fa fa-id-card sub-icon"></i><text class="nav-sub-title">All Personal Profiles</text></a></li>
	
	Link to current user personal profile
	
		<li>
			@Html.ActionLink("My Profile", "Details", "PersonalProfiles", new { id = User.Identity.GetUserId() }, htmlAttributes: new { @class = "removelinkdefault dropdown-links"})
		</li>
		
	Index View
	
		<div class="card-header">
			<h4 class="card-text1 text-center"> @Html.DisplayFor(modelItem => item.Person.FirstName) @Html.DisplayFor(modelItem => item.Person.LastName)</h4>
		</div>
		<div class="card-body">
			<table align="center">
				<tr>
					<td>
						<p class="card-text text-center">@Html.DisplayNameFor(model => model.AboutMe)</p>
						<p class="card-text1 text-center"> @Html.DisplayFor(modelItem => item.AboutMe)</p>
					</td>
				</tr>
				<tr>
					<td>
						<p class="card-text text-center">@Html.DisplayNameFor(model => model.TagLine)</p>
						<p class="card-text2 text-center"> @Html.DisplayFor(modelItem => item.TagLine)</p>
					</td>
				</tr>
				<tr>
					<td>@Html.Partial(AnchorButtonGroupHelper.PartialView, AnchorButtonGroupHelper.GetEditDetailsDelete(item.ProfileID.ToString()))</td>
				</tr>
			</table>
		</div>
	
## LogoProfileStyling
This story I had to remove the banner, place an image as the background, and have the logo and profile name and image display correctly.
	
	Layout
	
		<div id="header-nav">
            <header class="container">
                @*<img id="site-header">*@
                <div class="card-img-overlay" id="left-half">
                    <a id="header-title" href="@Url.Action("Dashboard","Home")"><img src="~/Content/images/newlogo.png" alt="" /></a>
                </div>
                
                <div class="card-img-overlay d-flex justify-content-center" id="right-half">
                    <div class="dropdown textcolor">
                        <button class="navbar-toggler btn" type="button" data-toggle="dropdown">
                            Welcome @ViewData["DisplayName"] <img id="ProfileImg" src="@Url.Action("Photo", "Manage" , new { UserId=User.Identity.GetUserId() })" height="48" width="48" />
                        </button>
                        <div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                            @Html.Partial("_LoginPartial")
                        </div>
                    </div>
                </div>
            </header>
        </div>
		
	CSS	
	
		#left-half {
			position: relative;
			left: 0px;
			/*float:left;*/
			grid-column:1;
			width: 50%;
		}

		#right-half {
			position: relative;
			/*float: right;*/
			grid-column: 2;
			left: 50%;
			right: 0;
			width: 300px;
			align-items: center;
			border-radius: 40px;
			margin-top: 20px;
			margin-bottom: 20px;
			background-color: #908AB6;
			padding:0px;
		}

*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*