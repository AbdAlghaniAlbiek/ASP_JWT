# Sending Emails

* [Importance of sending emails](#importance-of-sending-emails)
* [Setup](#setup)
* [HTML body](#html-body)

  
## Importance of sending emails
When a user register from android, windows, ios or web application, should have a verification process by sending an email from the backend web application to him and complete the registeration process that he started 
or if he finish registeraiton process, May backend web app can send a congratulation email to him.

## Setup
Please follow these steps:
1. Install two these packages 
   1. `MailKit`
   2. `MimeKit` 
2. Go to `appsettings.json` file add this (Every smtp provider (@outlook, @gmail, @live ...) has different configuration you should aware about that)
```csharp
"MailSettings": {
    "Email": "example@live.com",
    "DisplayName": "Name of sender this email",
    "Password": "P@ssword123",
    "Host": "smtp-mail.outlook.com",
    "Port": 587
}
```
3. Make folder `Settings` and add class `MailSettings`
```csharp
public class MailSettings
{
    public string Email { get; set; }

    public string DisplayName { get; set; }

    public string Password { get; set; }

    public string Host { get; set; }

    public int Port { get; set; }
}
```

4. To deserilaize mail settings that we describe it in `appsettings.json` to `MailSettings` and inject this class in every class's constructor we need this  line
```csharp
services.Configure<MailSettings>(Configuration.GetSection("MailSettings"));
```

5. We need add folder `Services` and add this class `MailService` and it's interface 'IMailService'
```csharp
public interface IMailingService
{
    Task SendEmailAsync(string mailTo, string subject, string body, IList<IFormFile> attachments = null);
}
```
```csharp
public class MailingService : IMailingService
{
    private readonly MailSettings _mailSettings;

    public MailingService(IOptions<MailSettings> mailSettings)
    {
        _mailSettings = mailSettings.Value;
    }

    public async Task SendEmailAsync(string mailTo, string subject, string body, IList<IFormFile> attachments = null)
    {
        var email = new MimeMessage
        {
            Sender = MailboxAddress.Parse(_mailSettings.Email),
            Subject = subject
        };

        email.To.Add(MailboxAddress.Parse(mailTo));

        var builder = new BodyBuilder();

        if (attachments is not null)
        {
            byte[] fileBytes;

            foreach (var file in attachments)
            {
                if (file.Length > 0)
                {
                    using var memoryStream = new MemoryStream();
                    await file.CopyToAsync(memoryStream);
                    fileBytes = memoryStream.ToArray();

                    builder.Attachments.Add(file.FileName, fileBytes, ContentType.Parse(file.ContentType));
                }
            }

            builder.HtmlBody = body;
            email.Body = builder.ToMessageBody();
            email.From.Add(new MailboxAddress(_mailSettings.DisplayName, _mailSettings.Email));

            using var smtp = new SmtpClient();
            smtp.Connect(_mailSettings.Host, _mailSettings.Port, SecureSocketOptions.StartTls);
            smtp.Authenticate(_mailSettings.Email, _mailSettings.Password);
            await smtp.SendAsync(email);

            smtp.Disconnect(true);
        }
    }
}
```

6. We need register this class and it's interface in DI (Dependency injection) sytem.
```csharp
services.AddTransient<IMailingService, MailingService>();
```

7. [Optional] To make test if it's working or not, you can copy and paste `sendEmail` action method
```csharp
[Route("api/[controller]")]
[ApiController]
public class MailingController : ControllerBase
{
    private readonly IMailingService _mailingService;

    public MailingController(IMailingService mailingService)
    {
        _mailingService = mailingService;
    }

    [HttpPost("send-email")]
    public async Task<IActionResult> SendEmail([FromForm] MailRequestDto dto)
    {
        await _mailingService.SendEmailAsync(dto.ToEmail, dto.Subject, dto.Body, dto.Attachments);

        return Ok();
    }
}
```


## HTML body
Assume that body of email is a HTML text, so let's applicate this scenario:
1. Add folder `Templates` and a html file inside it
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport"
          content="width=device-width" initial="text/html" ; charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta http-equiv="format-detection" content="date=no" />
    <meta http-equiv="format-detection" content="address=no" />
    <meta http-equiv="format-detection" content="telephone=no" />
    <meta http-equiv="x-apple-disable-message-reformatting" />

    <link href="https://fonts.googleapis.com/css?family=Multi:400,400i,700,700i"
          rel="stylesheet" />

    <title>Welcome</title>

    <style>
        /* Linked Styles */
        body {
            background: #eee;
            padding: 0 !important;
            margin: 0 !important;
            display: block !important;
            min-width: 100% !important;
            -webkit-text-size-adjust: none;
        }

        .email--background {
            background: #eee;
            padding: 10px;
            text-align: center;
        }

        .ematil-inner-container {
            padding: 0 5% 40px;
        }


        img {
            max-width: 100%;
            display: block;
            -ms-interpolation-mode: bicubic; /* Allow smoother rendering of resized image in Internet Explorer */
        }


        p {
            font-family 'Muli';
            font-size: 16px;
            line-height: 1.5;
            color: #a2a2a2;
            padding: 10px !important;
            margin: 0 !important;
        }

        .email--container, .pre-header {
            max-width 500px;
            background: #fff;
            margin: 0 auto;
            overflow: hidden;
            border-radius 5px;
        }

        .pre-header {
            background: #eee;
            color: #eee;
            font-size: 5px;
        }

        .cta {
            font-family 'Muli';
            display: inline-block;
            padding: 10px 20px;
            color: #fff;
            background: #66a6e6;
            text-decoration: none;
            letter-spacing: 2px;
            text-transform: uppercase;
            border-radius: 5px;
            font-size: 13px;
        }

        .footer-junk {
            padding: 20px;
            font-size: 10px;
        }

            .footer-junk a {
                color: #bbb;
            }
    </style>
</head>
<body class="body">
    <div class="email--background">
        <div class="email--container">
            <img src="https://yt3.ggpht.com/ytc/AAJvwnj2UAnA8Kz84c0lK7P8cGorQzxZuf4oFjKQ5dsW=s900-c-k-c0x00ffffff-no-rj" alt="DevCreed" />

            <div class="ematil-inner-container">
                <p>Welcome, [username]</p>
                <p>You are currently registered using</p>
                <p>[email]</p>
                <a href="https://www.youtube.com/c/DevCreed/" class="cta">Click here</a>
            </div>

        </div>

        <div class="footer-junk">
            <a href="#">Unsubscribe</a>
        </div>

    </div>
</body>
</html>
```

3. Let's assume that we have this controller `MailingContoller`, simply you can copy and paste the code inside `SendHTMLEmail` action method as it is and test by yoursef.
```csharp
[Route("api/[controller]")]
[ApiController]
public class MailingController : ControllerBase
{
    private readonly IMailingService _mailingService;
     
    public MailingController(IMailingService mailingService)
    {
        _mailingService = mailingService;
    }

    [HttpPost("send-html-email")]
    public async Task<IActionResult> SendHTMLEmail([FromBody] WelcomeRequestDto dto)
    {
        var filePath = $"{Directory.GetCurrentDirectory()}\\Templates\\EmailTemplate.html";
        var streamReader = new StreamReader(filePath);

        var mailText = streamReader.ReadToEnd();
        streamReader.Close();

        mailText = mailText.Replace("[username]", dto.UserName).Replace("[email]", dto.Email);

        await _mailingService.SendEmailAsync(dto.Email, "Welcome to our channel", mailText);

        return Ok();
    }
}
```


