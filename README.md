# LiveProject
Live Project I worked on at The Tech Academy. 
<div>
<h2>Introduction</h2>
I was assigned to work for two weeks on a Live Project, where a DevOps team worked on a MVC Web Application for two weeks. The project was already well underway, so this was a great chance for me to experience studying, adding to, and cleaning up code that people had previously worked on. 
</div>

<div>  
<h2>Back End Story</h2>
</div>

<div>
	<h3>Linking Display to Database</h3>

<div>My first task was to sort the archive by past, current and future productions. To do this automatically as not have to manually as each season changes, I linked the archive to the database listing all the productions.</div>

<div><strong>View: Archive.cshtml</strong></div>

    //Current Productions Referencing Database
    <div id="current-productions">
    <div class="card-deck bg-black">
	@{
	var currentSettings = AdminSettingsReader.CurrentSettings(); // Grabs AdminSettings JSON
    int currentSeason = currentSettings.current_season; // Initialize currentSeason and set using "current_season" from JSON
	
	foreach (var item in Model)
        {
            if (item.Season == currentSeason)
            {
                <div class="card bg-black">
                    @{
                    if (item.DefaultPhoto != null)
                    {
                        <a href="@Url.Action("Details", "Productions", new { id = item.ProductionId })">
                            <img class="card-img-top production-index-img mt-5 bg-black" src="@Url.Action("DisplayPhoto", "Photo", new { id = item.DefaultPhoto.PhotoId })" alt="@Url.Action(item.Title)"}></a>
                    }
                    else
                    {
                        <a href="@Url.Action("Details", "Productions", new { id = item.ProductionId })">
                            <img class="card-img-top production-index-img mt-5 bg-black" src="~/Content/Images/Unavailable.png" alt="@Url.Action(item.Title)"></a>
                    }
                    }
                    <div class="card-body production-index-card-inner justify-content-center">
                        <h5 class="card-title">
                            @item.Title
                            <span class="badge badge-pill badge-primary">Season: @item.Season</span>
                        </h5>
                        <p class="card-text">@item.Description</p>
                    </div>
                    <div class="card-footer align-bottom">
                        <button type="button" class="btn btn-block btn-primary" disabled>Get Tickets</button>
                    </div>
                </div>
            }
        }
        }
    </div>
</div>

I repeated this process for linking the database to the production listings for the past and future productions.
</div>

<h3>Add PhotoId property to CastMember class</h3>

My mission was to add the property ‘PhotoId’ to CastMember class to show the cast member’s photos and get it to function properly.
The first thing to do was to add the property to the ‘CastMember’ class and table in the database.

<div><strong>Model: CastMember.cs</strong>

        public class CastMember
        {
            public int CastMemberID { get; set; }           // cast member primary key
            public string Name { get; set; }                // cast member display name
            public string Bio { get; set; }                 // cast member biographical excerpt
            public int PhotoId { get; set; }                // Id of cast member photo
            public PositionEnum MainRole { get; set; }      // position at theater
            public bool CurrentMember { get; set; }         // active permanent or temporary cast member
            public string CastMemberPersonID { get; set; }  // user ID for cast member
            public int? DebutYear { get; set; }             // first year that cast member joins theater
        }
</div>

<div>
Next, I had to make sure that the CastMembersController would create the photo being uploaded by the user. So I pulled the ‘CreatePhoto’ method from the PhotoController that saves the photo to the database:

        public static int CreatePhoto(HttpPostedFileBase file, string title)
        {
            var photo = new Photo();
            using (ApplicationDbContext db = new ApplicationDbContext())
            {
                photo.Title = title;
                Image image = Image.FromStream(file.InputStream, true, true);
                photo.OriginalHeight = image.Height;
                photo.OriginalWidth = image.Width;
                var converter = new ImageConverter();
                photo.PhotoFile = (byte[])converter.ConvertTo(image, typeof(byte[]));
                db.Photo.Add(photo);
                db.SaveChanges();
                return photo.PhotoId;
            }
        }
</div>

<div>
and plugged it into:
<strong>CastMembersController.cs</strong>

        public ActionResult Create([Bind(Include = "CastMemberID,Name,Bio,Photo,MainRole,CurrentMember,CastMemberPersonId,DebutYear")] CastMember castMember, HttpPostedFileBase file)
        {
     ~~code~~
            if (ModelState.IsValid)
            {
	   // Assign cast member a PhotoId
                if (file != null && file.ContentLength > 0)
                {
                    castMember.PhotoId = PhotoController.CreatePhoto(file, castMember.Name);
  	            }
     ~~code~~
                return RedirectToAction("Index");
            }
            else 
            {
                ViewData["dbUsers"] = new SelectList(db.Users.ToList(), "Id", "UserName");
            }
            return View(castMember);
        }
</div>

<div>
<h2>Front End Story</h2>
</div>

<div>
I also needed to have the user be able to interact with this new PhotoId property and preferably be able to preview their photo before they uploaded it, so I applied JQuery.
<strong>View: Create.cshtml</strong>

    @using (Html.BeginForm("Create", "CastMembers", FormMethod.Post, new { enctype = "multipart/form-data" }))
    {
        @Html.AntiForgeryToken()
        <div class="formContainer2 ">
            <div class="form-horizontal">
                <h4 class="formHeader">CastMember</h4>
                <hr />
                <div class="inputBox2">
                    @Html.ValidationSummary(true, "", new { @class = "text-danger" })
    ~~code~~
                    <div class="form-group">
                        @Html.LabelFor(model => model.PhotoId, htmlAttributes: new { @class = "control-label col-md-2 inputLabel",  })
                        <div class="col-md-10 formBox">
                            @Html.TextBoxFor(model => model.PhotoId, new { type = "file", Name = "file", Class = "fileSelect", onchange = "PreviewImage(event)" })
                                <img id="preview" />                        
                            @Html.ValidationMessageFor(model => model.PhotoId, "", new { @class = "text-danger" })
                        </div>
                    </div>
    ~~code~~
                </div>
            </div>
        </div>
    }
    <div>
        @Html.ActionLink("Back to List", "Index")
    </div>

    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
    <script type="text/javascript">
        function PreviewImage(event) {
            var output = document.getElementById('preview');
            output.src = URL.createObjectURL(event.target.files[0]);
        };
    </script>
</div>
<div>
I applied this PhotoId property to the rest of the CRUD operations and web application.
</div>

