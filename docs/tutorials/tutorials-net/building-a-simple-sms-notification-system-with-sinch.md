---
title: "Building a simple SMS notification system with Sinch"
excerpt: "This tutorial will show you how to build a simple notification system by using Sinch. This will be a system I built for the Western Grands. Every year there are three main events in Quarter Midget racing, one for each region and one dirt event."
---
A few months back, my son and I started to race [quarter midgets](https://en.wikipedia.org/wiki/Quarter_Midget_racing). It’s been a lot of fun, and a steep learning curve. Believe me, getting perfectly aligned cells without using tables in HTML is pretty easy compared to tuning a racing chassis\! It’s also amazing to meet the people that put in the hard work and their heart to make racing (somewhat) affordable for kids. Anyway, everyone is encouraged to contribute to the club with their skills - in my case, coding is a skill I am at least somewhat competent in. In this article I’ll show you a simple notification system I built for the Western Grands. Every year there are three main events in Quarter Midget racing, one for each region and one dirt event. Our club [Tri Valley Quarter Midget Association](https://www.tvqma.org/) has the honor of hosting the events this time around.
![qmcar.jpg](images/34030b1-qmcar.jpg)

This year, 250 cars and about 125 drivers will come from all over the Western States to race at our track in 17 different classes, for three intensive days. To manage all the races and drivers, and make sure everyone is in the right place at the right time, there’s a lot of logistics to take care of. To give you an idea of the schedule, 2 days alone are taken up with parking trailers, then every car needs to be checked for the correct fuel, inspected for safety, get weighed, have lap times measured and last but not least, get the cars and kids out on the track. Then each kid has practice laps, qualifying laps, heats, Lower Mains and the A mains. This is usually managed by a single person announcing over a PA system where people need to be at certain times. This year, the event is going to be too big for this to be effective. If someone is in their trailer, there’s no way they’ll be able to hear the announcer during the Grands. To solve this problem, I suggested a simple SMS system for the Tower and Pit stewards, enabling them to send SMS to communicate announcements. We believe this system has the potential to help people be on time, and also give them the confidence to relax and have fun\! After all, that’s what it’s all about\!

## Prerequisites

This article assumes that you are familiar with .net core and ASP.net MVC patterns, you will also need an account and a phone number with Sinch. The solution is kind of ready to run, you just need to add a valid connection string. There is more code in the repo in this article, but I will talk about the Sinch specific parts.

## Time to build

So the basic idea is that the track official who would usually announce something over a PA system, also has access to a tool to send a quick SMS out to everyone containing the same message. Due to regulations in the US, we are going to use [Toll-FreeNumbers](https://www.clxcommunications.com/products/toll-free-numbers/) to send SMS to ensure high throughput and no spam filter. If you live in Europe, you can pick any number you like in most countries.

First we needed to collect phone numbers from racers, we managed this by advertising on social media, and having signs around the track asking people to send an SMS containing ‘START’ to +1 888-851-0949. Once a text was sent, the number was added to the database.

The next thing we needed was a way for track officials to send out messages. In this case, we had two possibilities: 1. Via a website 2. Via a whitelist of numbers with the ability to send SMS to the number listed above
![flowchart.png](images/a0a6699-flowchart.png)

### Managing signups via SMS

I bought a number in the [portal](https://portal.sinch.com/#/numbers) (Yeah, I know we should have way more countries in stock, it’s coming. For now mail me if you need a particular country), created an app and assigned the number a webhook url to receive SMS. During development I always use the awesome tool [ngrok](doc:getting-a-second-number-and-testing-sinch-callbackswebhooks-with-ngrok) during development.
![dashboardcallback.png](images/7787cde-dashboardcallback.png)

Next I needed to add a WebApi controller to handle all incoming SMS.

*SMSController.cs*

```csharp
[Produces("application/json")]
public class SMSController : Controller {
    private readonly SMSSender _smsSender;
    private readonly ApplicationDbContext _dbContext;
    private readonly IConfiguration _configuration;

    public SMSController(ApplicationDbContext dbContext, SMSSender smsSender, IConfiguration configuration) {
       _smsSender = smsSender;
       _dbContext = dbContext;
       _configuration = configuration;
    }

    [Route("/SMS")]
    public async Task<OkResult> Post([FromBody] IncomingMessageEvent model) {
        var fromNumber = "+" + model.From.Endpoint;
        Subscriber subscriber = _dbContext.Subscribers.FirstOrDefault(m => m.Number == fromNumber);
        if (model.Message.Trim().ToLower() == "start" || model.Message.Trim().ToLower() == "unstop") {
            if (subscriber == null) {
                subscriber = new Subscriber {
                    Number = fromNumber
                };
                _dbContext.Subscribers.Add(subscriber);
                await _dbContext.SaveChangesAsync();
            }
            await _smsSender.SendSMS(new Message { MessageContent = _configuration["Sinch:WelcomeMessage"] } , subscriber);
            return Ok();
        }

        if (model.Message.Trim().ToLower() == "stop") {
            if (_dbContext.Subscribers.Any(m => m.Number == fromNumber)) {
                _dbContext.Subscribers.Remove(_dbContext.Subscribers.First(m => m.Number == fromNumber));
                await _dbContext.SaveChangesAsync();
            }
            //add subscribper

            return Ok();
        }
        await _smsSender.SendSMS(new Message { MessageContent = "Sorry, we only support Start and stopm if you have any questions please contact TVQMA" }, subscriber);
        return Ok();
    }
}
```

There are a few things to mention here. Firstly I wanted to reach the “Start” and “Stop” keyword, the unstop command kicks in if you start, then send a stop message. If this happens you’ll need to send an unstop message to re-enable sms from that number.

In the start command I checked to make sure that the subscriber didn’t already exist, if they weren’t already in the database I added them, and then finally sent out the welcome message. I opted for sending the message even if the subscriber exists, since it’s a command the program still understands. **One Gotcha here, we send you the number without + in e 164 format, but we require you to send it with a + to make sure its a country code hence the var fromNumber = “+” + model.From.Endpoint;**

I also wanted to support stop, so that people could remove themselves, and also provide somewhat meaningful feedback if something was sent in that we didn’t understand.

### Subscriber data class

The subscriber data class keeps track of the people that send in a start SMS, it’s using Entity framework. In the github repo you will also see that I scafolded the whole class to provide the club with a crude admin of subscribers.

*Susbscriber.cs*

```csharp
public class Subscriber {
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int SubscriberId { get; set; }
    public string Number { get; set; }
}
```

And then I added it as a db set to the ApplicationDbContext as a DBset.

*ApplicationDbContext.cs*

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser> {
    public DbSet<Message> Messages { get; set; }
    public DbSet<Subscriber> Subscribers{ get; set; }
    public DbSet<SendLog> SendLogs { get; set; }

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        // Customize the ASP.NET Identity model and override the defaults if needed.
        // For example, you can rename the ASP.NET Identity table names and more.
        // Add your customizations after calling base.OnModelCreating(builder);
    }
    }
```

Then the SMS is sent through an SMSsender service that is added in Startup.cs - I will cover that a bit later. As you can see, I use the Sinch Nuget package to help me out: <https://www.nuget.org/packages/Sinch.ServerSdk/> While not necessary, it sure makes it easier when it comes to signing requests.

*SMSSender.cs*

```csharp
public class SMSSender {
        private ApplicationDbContext _context;
        private readonly IConfiguration _configuration;
        private SinchConfig _sinchConfig;

        public SMSSender(ApplicationDbContext context, IConfiguration configuration) {
            _context = context;

            _configuration = configuration;
            _sinchConfig = context.SinchConfig.FirstOrDefault();
        }
        public async Task SendSMS(Message message, Subscriber to) {
            var smsApi = SinchFactory.CreateApiFactory(_sinchConfig.Key, _sinchConfig.Secret).CreateSmsApi();
            var messageid = await smsApi.Sms(to.Number,
                    message.MessageContent + "\n\n" +
                    _sinchConfig.MarketingFooter)
                .WithCli(_sinchConfig.SinchNumber).Send();
            _context.SendLogs.Add(new SendLog() {
                DateSent = DateTime.UtcNow,
                MessageId = message.MessageId,
                SubscriberId = to.SubscriberId,
                SinchMessageId = messageid.MessageId.ToString()
            });
            await _context.SaveChangesAsync();

        }
    }
```

This service creates an SMS API, adds the footer marketing text, and logs the SMS message to a send log, so if we needed to implement status checks at a later stage, we would be able to. In startup.cs in the ConfigureServices, I added a line so dependency injection could take care of this whenever I needed to send an SMS message.

```csharp
services.AddTransient<SMSSender>();
```

## Sending SMS notifications.

I wanted to store each message sent, and also keep a log of who messages were sent to, so I added two more classes to support this.

*Message.cs*

```csharp
public class Message {
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int MessageId { get; set; }
    public String MessageContent { get; set; }
    public DateTime DateSent { get; set; }
    public virtual IEnumerable<SendLog> Logs { get; set; }
}
```

and *SendLog.cs*

```csharp
public class SendLog {
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int SendLogId { get; set; }
    public int SubscriberId { get; set; }
    public int MessageId { get; set; }
    public string SinchMessageId { get; set; }
    public DateTime DateSent { get; set; }
    public DateTime DateDelivered { get; set; }
    public string Status { get; set; }
    public virtual Subscriber Subscriber { get; set; }
    }
```

I wanted to show this in a dashboard style layout, so a ViewModel was also needed.

*DashboardModel.cs*

```csharp
public class DashboardModel {
    public int SubscriberCount { get; set; }
    public string Message { get; set; }
    public List<Message> LastMessages { get; set; }
}
```

Finally a Controller and a View.

*MessageContoller.cs*

```csharp
public class MessagesController : Controller {
    private readonly ApplicationDbContext _context;
    public MessagesController(ApplicationDbContext context) {
        _context = context;
    }

    [Route("/dashboard")]
    [HttpGet]
    public IActionResult Dashboard()  {
        var model = new DashboardModel();
        model.SubscriberCount = _context.Subscribers.Count();
        model.LastMessages = _context.Messages.OrderByDescending(m => m.DateSent).Take(10).ToList();
        model.Message = "";
        return View(model);
    }

        [Route("/dashboard")]
        [HttpPost]
        public async Task<IActionResult> Dashboard(DashboardModel model) {
            if (ModelState.IsValid) {
                await CreateAndSendMessage(new Message {
                    MessageContent = model.Message,
                    DateSent = DateTime.UtcNow
                });
            }

            model.SubscriberCount = _context.Subscribers.Count();
            model.LastMessages = _context.Messages.OrderByDescending(m => m.DateSent).Take(10).ToList();
            model.Message = "";
            return View(model);
        }
        private async Task CreateAndSendMessage(Message message) {
            message.DateSent = DateTime.UtcNow;
            _context.Add(message);
            await _context.SaveChangesAsync();
            var subscribers = await _context.Subscribers.ToListAsync();
            foreach (var s in subscribers) {
                await _smsSender.SendSMS(message, s);
            }
        }
}
```

I wanted to make it super easy for officials to send a quick message. Getting drivers to staging is a critical step of the race, if you don’t show up on time for your race you will miss it.

*Dashboard.cshtml*

```
@model TVQMANotifications.Models.DashboardModel
@{
    ViewBag.Title = "TVQMA Dashboard";
}

<H1>Send an SMS to 844-287-2483 with the the word START in it.</H1>
<div class="row">
    <div class="col-md-6">
        <h2>Send a new notification</h2>
        <form asp-action="Dashboard">
            @*<input type="hidden"  value="Jr Novice to Staging"/>*@
            <input type="submit" asp-for="Message" value="Jr Novice to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Sr Novice to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Jr Animal to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Sr Animal to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Hvy Animal to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Jr Honda to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Sr Honda to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Hvy Honda to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Jr Stock to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Mod  to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Lt 160 to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Hvy 160 to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="B to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="AA/Modified World to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Jr Half to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Lt World Formula to staging" class="btn btn-default" />
            <input type="submit" asp-for="Message" value="Hvy World Formula to staging" class="btn btn-default" />
        </form>        
        <form asp-action="Dashboard">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Message" class="control-label"></label>
                <textarea asp-for="Message" class="form-control" ></textarea>
                <span asp-validation-for="Message" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Send" class="btn btn-default" />
            </div>
        </form>        
    </div>
    <div class="col-md-6">
        <h2>Number of Subscribers @Model.SubscriberCount</h2>
        <table class="table">
            <thead>
            <tr>
                <th colspan="2">
                    Latest messages
                </th>
                <th>
                </th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            @{
                var i = 0;
            }
            @foreach (var item in Model.LastMessages)
            {
                i =++ i;
                <tr  class="@(i == 1 ? "sucess" : "")" style="background: @(i == 1 ? "#dff0d8" : "")">
                    <td >
                        @item.DateSent.AddHours(-7)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.MessageContent)
                    </td>

                    <td>
                        <a asp-action="Details" asp-route-id="@item.MessageId">Details</a>
                    </td>
                </tr>
            }
            </tbody>
        </table>

    </div>
    <div class="row">
        <div class="col-md-6">
            To unsubscibe sent stop to same number
        </div>
    </div>
</div>
```

That’s pretty much it when it comes to the SMS and Sinch functionality, in the solution you will see some user handling and a few pages to deal with messages that is standard aspnet stuff. Clone this and let me know if you have any questions.


<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/tutorials/net/building-a-simple-sms-notification-system-with-sinch.md"><span class="fab fa-github"></span>Edit on GitHub</a>