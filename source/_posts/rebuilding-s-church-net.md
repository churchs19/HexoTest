title: Rebuilding s-church.net
date: 2011/12/16 16:00:43
alias: blog/455/
tags:
- Software
- ASP.NET
- jQuery
- HTML5
- CSS3
- Amazon S3
- Windows Live
- C#
photos:
- /journal_images/Windows-Live-Writer/792cfd3a63f5_8BDB/screensaver_3.jpg|Screensaver
---
![Shane.Church.Web Project Structure](/journal_images/Windows-Live-Writer/792cfd3a63f5_8BDB/Shane.Church.Web.ProjectStructure_1.png "Shane.Church.Web Project Structure")

Along with the posts that I’ve been getting back into making recently, I also spent quite a bit of time recently redeveloping the infrastructure of s-church.net.  I rebuilt the site from the ground up using [ASP.NET MVC 3](http://www.asp.net/mvc), [HTML5](http://www.html5rocks.com/en/), CSS 3, and [jQuery](http://www.jquery.com) and I’d like to take some time to describe what I did from a technical standpoint.

The first part of what I did was to create a new ASP.NET MVC 3 project in Visual Studio.  I then analyzed my existing site and and created MVC controllers for each of the main menu items that you see above.  I rebuilt the main _Layout.cshtml master page for the site using HTML5 and the ASP.NET MVC 3 Razor view engine as seen in  Master Layout code snippet below.  I take advantage of the new HTML5 semantic tags like <span style="color: blue"><</span><span style="color: maroon">header</span><span style="color: blue">></span>, <span style="color: blue"><</span><span style="color: maroon">nav</span><span style="color: blue">></span>, <span style="color: blue"><</span><span style="color: maroon">section</span><span style="color: blue">></span>, <span style="color: blue"><</span><span style="color: maroon">article</span><span style="color: blue">></span>, and <span style="color: blue"><</span><span style="color: maroon">footer</span><span style="color: blue">></span>.  The <span style="color: blue"><</span><span style="color: maroon">header</span><span style="color: blue">></span>, <span style="color: blue"><</span><span style="color: maroon">nav</span><span style="color: blue">></span>, and <span style="color: blue"><</span><span style="color: maroon">footer</span><span style="color: blue">></span> tags are relatively straightforward in their usage, but the <span style="color: blue"><</span><span style="color: maroon">section</span><span style="color: blue">></span> and <span style="color: blue"><</span><span style="color: maroon">article</span><span style="color: blue">></span> tags are less obvious.  In this site, I use <span style="color: blue"><</span><span style="color: maroon">section</span><span style="color: blue">></span> as a container for various pieces of the layout.  Each blog post or photo item container is an <span style="color: blue"><</span><span style="color: maroon">article</span><span style="color: blue">></span> in the semantic markup.

```xml
<div class="page">
        <header>
            <div id="title">
                <h1>@ViewBag.Header</h1>
            </div>
        </header>
        <nav>
            <ul class="ssc_nav">
                <li>@Html.ActionLink("Home", "Index", "Home")</li>
                <li><a href="@Url.Content("~/Resume.pdf")">Resume</a></li>
                <li>@Html.ActionLink("Blog", "Index", "Blog")</li>
                <li>@Html.ActionLink("Photo Album", "Index", "PhotoAlbum")</li>
                <li>
                    <a href="#">Software</a>
                    <div class="ssc_sub_nav">
                        @Html.ActionLink("Windows 7 Gadgets", "Win7Gadgets", "Software")
                        @Html.ActionLink("Windows Mobile Smartphone", "WMSmartphone", "Software")
                        @Html.ActionLink("Windows Mobile Pocket PC", "WMPocketPC", "Software")
                        @Html.ActionLink("Windows", "MSWindows", "Software")
                        @Html.ActionLink("Pocket PC", "MSPocketPC", "Software")
                        @Html.ActionLink("Palm OS", "Palm", "Software")
                    </div>
                </li>
                <li>
                    @Html.ActionLink("Reading List", "Index", "ReadingList")
                </li>
                @if (Request.IsAuthenticated && User.IsInRole("Admin"))
    {
                <li>@Html.ActionLink("Admin", "Index", "Admin")</li>
    }
            </ul>
        </nav>
        <section id="main">
            <div class="main_container">
                @RenderBody()
            </div>
            <div class="side_nav">
                <div class="container">
                    <img src="@Url.Content("~/Content/Images/side_nav_photo.jpg")" alt="Shane Church, Andrea Love, and Ethan Love-Church" class="side_photo" />
                    <h3>Contact</h3>
                    <a href="mailto:shane@s-church.net" class="logo_link">Email Me</a>
                    @RenderSection("SideNav", false)
                </div>
                @RenderSection("SideNavPostContainer", false)
            </div>
            <div class="clear">&nbsp;</div>
        </section>
        <footer>
            <div id="logindisplay">
                @Html.Partial("_LogOnPartial")
            </div>
            Copyright &copy; Shane Church 2011
        </footer>
    </div>
```

I built the menu hover and drop down behaviors myself using jQuery and some CSS.  The behavior is relatively straightforward and, as I’ve been learning more about HTML, CSS, and JavaScript in my role at [EffectiveUI](http://www.effectiveui.com), I found it very easy to implement the menu on my own as shown in the code snippet below.

```JavaScript
var menuTimeout = null;

$(document).ready(function () {
    $(".ssc_nav li").mouseenter(function () {
        var $this = $(this);
        $this.find(".ssc_sub_nav").slideDown(200, function () {
            menuTimeout = window.setTimeout(function () {
                $this.find(".ssc_sub_nav").slideUp(200);
            }, 500);
        });
    });

    $(".ssc_sub_nav a").mouseenter(function () {
        window.clearTimeout(menuTimeout);
        menuTimeout = null;
    });

    $(".ssc_sub_nav a").mouseleave(function () {
        window.clearTimeout(menuTimeout);
        menuTimeout = null;
        var $this = $(this);
        menuTimeout = window.setTimeout(function () {
            $this.parent().slideUp(200);
        }, 500);
    });
});
```

All of the styling of the new site is using CSS 3 with minimal use of images for item backgrounds.  This significantly improves the download size and speed of the individual pages by reducing the number of HTTP requests for images and allowing the browser to do the rendering work.  I’m using [Eric Meyer’s Reset CSS](http://meyerweb.com/eric/tools/css/reset/) to reset all of the CSS settings across all browsers before applying my own styling to achieve consistency between all of the different browsers.  One annoyance to me in the new CSS 3 implementations is the addition of browser specific prefixes like -moz- for Firefox and -webkit- for Chrome and Safari and -ie- for Internet Explorer.  This requires me as the developer to know a lot more about the workings of each browser to achieve consistent results and I end up having the [W3Schools reference pages](http://www.w3schools.com) open constantly while developing.  For example, the gradient in the main menu is described by the CSS code below.

```css
ul.ssc_nav {
    height: 34px;
    background: #999999; /* Old browsers */
    background: -moz-linear-gradient(top, #999999 0%, #5B5B5B 100%); /* FF3.6+ */
    background: -webkit-gradient(linear, left top, left bottom, color-stop(0%,#999999), color-stop(100%,#5B5B5B)); /* Chrome,Safari4+ */
    background: -webkit-linear-gradient(top, #999999 0%,#5B5B5B 100%); /* Chrome10+,Safari5.1+ */
    background: -o-linear-gradient(top, #999999 0%,#5B5B5B 100%); /* Opera11.10+ */
    background: -ms-linear-gradient(top, #999999 0%,#5B5B5B 100%); /* IE10+ */
    filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#999999', endColorstr='#5B5B5B',GradientType=0 ); /* IE6-9 */
    background: linear-gradient(top, #999999 0%,#5B5B5B 100%); /* W3C */
    border-top: 1px solid #000000;
    border-bottom: 1px solid #000000;
    z-index: 200;
}
```

I wanted to make sure that I maintained all of the existing links which were in the format of /Blog.aspx?id=<id> even though I was switching the site to use a more SEO-friendly URL routing scheme of `/Blog/<id>`, so I added a route in Global.asax to my LegacyController which then executes a HTTP 301 Permanent Redirect to the new URL so that search engines and people with favorites marked for the old URLs will continue to work correctly.  The routing entry is as follows:

```cs
routes.MapRoute(
        "{*allaspx}",
        @"{id}.aspx",
        new { controller = "Legacy", action = "Index" });
```

and the PermanentRedirectResult MVC action is highlighted below.

```cs
public class PermanentRedirectResult: ActionResult
{
    //...

    public override void ExecuteResult(ControllerContext context)
    {
        if (_action == null || _controller == null)
        {
            context.HttpContext.Response.Status = "404 Not Found";
            context.HttpContext.Response.StatusCode = 404;
        }
        else
        {
            context.HttpContext.Response.Status = "301 Moved Permanently";
            context.HttpContext.Response.StatusCode = 301;
            context.HttpContext.Response.AppendHeader("Location", ((Controller)(context.Controller)).Url.Action(_action, _controller, _routeValues));
        }
    }
}
```

![Screensaver](/journal_images/Windows-Live-Writer/792cfd3a63f5_8BDB/screensaver_3.jpg "Screensaver")

During the transition, I also converted my photo album from local file storage to use [Amazon’s S3 service](http://aws.amazon.com).  This change required me to implement a simple controller to use Amazon’s AWS SDK and then I added an ASP.NET MVC controller method in my photo album controller to actually respond with the image from the Amazon service as shown below.  This conversion also required me to retool my custom screensaver that displays images from this site (shown to the left) and my [Windows Live Photo Gallery](http://explore.live.com) plugin for publishing images to the site to use the Amazon S3 service as well.

```cs
public ActionResult Image(long id)
{
    MySQLSiteModel model = new MySQLSiteModel();
    var photo = (from p in model.photo
                 join pg in model.page on p.LINK_ID equals pg.ID
                 where p.ID == id
                 select new PhotoViewModel() { ID = p.ID, CategoryID = pg.ID, CategoryName = pg.DISPLAY_NAME, Caption = p.CAPTION, File = p.FILE, UpdatedDate = p.UPDATED_DATE }).FirstOrDefault();

    try
    {
        S3Controller amazonController = new S3Controller();

        Stream data = amazonController.ReadObject("photos", photo.CategoryID.ToLower() + "." + photo.File.Substring(photo.File.LastIndexOf('/') + 1).ToLower());

        return File(data, "image/jpeg");
    }
    catch
    {
        return new HttpStatusCodeResult(404);
    }
}
```

As you can see illustrated in the snippet above, I have also moved my database connection code to use the Microsoft Entity Framework using the MySQL .NET provider.  The database connections are then handled in code using LINQ-to-Entities, significantly reducing the amount of raw SQL code that I needed.  I did run into some issues with the MySQL provider not supporting some of the standard LINQ operations like .Take() consistently, though I don’t have a good code example for that since I came up with some workarounds.

I also implemented the Metaweblog API in order to facilitate using [Windows Live Writer](http://explore.live.com) to publish to the blog.  I implemented this using the [XML-RPC.Net](http://xml-rpc.net/) library as a ASP.NET generic handler.  The interface is shown below.

```cs
public interface IMetaWeblog
{
    #region MetaWeblog API

    [XmlRpcMethod("metaWeblog.newPost")]
    string AddPost(string blogid, string username, string password, Post post, bool publish);

    [XmlRpcMethod("metaWeblog.editPost")]
    bool UpdatePost(string postid, string username, string password, Post post, bool publish);

    [XmlRpcMethod("metaWeblog.getPost")]
    Post GetPost(string postid, string username, string password);

    [XmlRpcMethod("metaWeblog.getCategories")]
    CategoryInfo[] GetCategories(string blogid, string username, string password);

    [XmlRpcMethod("metaWeblog.getRecentPosts")]
    Post[] GetRecentPosts(string blogid, string username, string password, int numberOfPosts);

    [XmlRpcMethod("metaWeblog.newMediaObject")]
    MediaObjectInfo NewMediaObject(string blogid, string username, string password,
        MediaObject mediaObject);

    #endregion

    #region Blogger API

    [XmlRpcMethod("blogger.deletePost")]
    [return: XmlRpcReturnValue(Description = "Returns true.")]
    bool DeletePost(string key, string postid, string username, string password, bool publish);

    [XmlRpcMethod("blogger.getUsersBlogs")]
    BlogInfo[] GetUsersBlogs(string key, string username, string password);

    [XmlRpcMethod("blogger.getUserInfo")]
    UserInfo GetUserInfo(string key, string username, string password);

    #endregion
}
```

One final piece to note is the use of jQuery to make the pages more interactive.  For each major section of the site, I wrote the code as a [jQuery Plugin](http://docs.jquery.com/Plugins/Authoring) in order to encapsulate the code for each section.  This is a technique that we commonly use at [EffectiveUI](http://www.effectiveui.com) to make it easier for our clients to maintain the code after a project is complete.  An example from the Photo Album plugin that I wrote is shown below.

```JavaScript
//jquery.PhotoAlbum.js
//Copyright © Shane Church 2011

(function ($) {
    var methods = {
        init: function () { },
        loadPage: function (options) {
            var defaults = {
                href: "",
                callback: function () { }
            };

            var opts = $.extend(defaults, options);

            var $this = $(this);

            $.ajax({
                url: opts.href,
                type: "GET",
                dataType: 'html',
                success: function (data) {
                    var $data = $(data);

                    $(".photos_container").empty();
                    $(".photos_container").append($data.find(".photos_container").children());

                    $(".pager").empty();
                    $(".pager").append($data.find(".pager").children());

                    opts.callback();
                },
                error: function (request, status, error) {
                }
            });
        },
        showPhoto: function (options) {
            var defaults = {
                href: "",
                title: "",
                category: "",
                file: "",
                callback: function () { }
            };

            var opts = $.extend(defaults, options);

            var $content = $("#modal-view-photo");
            $content.find(".image img").attr("src", opts.file).attr("alt", opts.category + " - " + opts.title);
            $content.find("h4").text(opts.title);
            $content.find("h5").text(opts.category);
            $content.find("a.close").unbind("click").click(function () {
                $().HideModal({ strId: "#modal-view-photo" });
                return false;
            });

            var options = {
                strId: "#modal-view-photo"
            };

            $().ShowModal(options);
        }
    };

    $.fn.PhotoAlbum = function (method) {
        if (methods[method]) {
            return methods[method].apply(this, Array.prototype.slice.call(arguments, 1));
        } else if (typeof method === 'object' || !method) {
            return methods.init.apply(this, arguments);
        } else {
            $.error('Method ' + method + ' does not exist on jQuery.PhotoAlbum');
        }
    };

})(jQuery);
```

As you can see, this was a pretty major rearchitecture of the entire site, but it gave me the opportunity to put a lot of the new techniques that I’ve learned over the last year into practice as well as build an updated portfolio site.  I hope this detailed tour of the site proves useful.  Feel free to comment or contact me directly with any comments or questions!