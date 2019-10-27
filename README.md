# VideoMetrics
A project to record the usage of Vimeo and YouTube videos on a web page.

### 1. Database configuration
```
CREATE SCHEMA `videometrics` ;
CREATE USER 'videomentrics'@'localhost' IDENTIFIED BY 'YOURPASSWORDHERE';
GRANT ALL PRIVILEGES ON videometrics.* TO 'videomentrics'@'localhost';
```
### 2. Configure the software
Copy the file .env.Example to .env

2.1 Change the value of APP_URL to that appropriate for your hosting environment
```
APP_URL=http://127.0.0.1:8000
```

2.2 Set the Database parameters
```
#-- ===================================================================== --#
#-- Database connection parameters                                        --#
#-- ===================================================================== --#
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=videometrics
DB_USERNAME=videometrics
DB_PASSWORD=YOURPASSWORDHERE
```

2.3 Set the application key
In a command prompt in the project directory (e.g. videometrics\Service\VideoMetrics) run the following commands:
```
php artisan key:generate
composer install
php artisan migrate
```

### 3. Configure your web pages
You will need to add the following to your webpage:
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/modernizr/2.8.3/modernizr.js" integrity="sha256-ffw+9zwShMev88XNrDgS0hLIuJkDfXhgyLogod77mn8=" crossorigin="anonymous"></script>
<script src="https://player.vimeo.com/api/player.js"></script>

<script>

    var BrowserDetect = {
        init: function () {
            this.browser = this.searchString(this.dataBrowser) || "Other";
            this.version = this.searchVersion(navigator.userAgent) || this.searchVersion(navigator.appVersion) || "Unknown";
        },
        searchString: function (data) {
            for (var i = 0; i < data.length; i++) {
                var dataString = data[i].string;
                this.versionSearchString = data[i].subString;

                if (dataString.indexOf(data[i].subString) !== -1) {
                    return data[i].identity;
                }
            }
        },
        searchVersion: function (dataString) {
            var index = dataString.indexOf(this.versionSearchString);
            if (index === -1) {
                return;
            }

            var rv = dataString.indexOf("rv:");
            if (this.versionSearchString === "Trident" && rv !== -1) {
                return parseFloat(dataString.substring(rv + 3));
            } else {
                return parseFloat(dataString.substring(index + this.versionSearchString.length + 1));
            }
        },

        dataBrowser: [
            {string: navigator.userAgent, subString: "Edge", identity: "MS Edge"},
            {string: navigator.userAgent, subString: "MSIE", identity: "Explorer"},
            {string: navigator.userAgent, subString: "Trident", identity: "Explorer"},
            {string: navigator.userAgent, subString: "Firefox", identity: "Firefox"},
            {string: navigator.userAgent, subString: "Opera", identity: "Opera"},
            {string: navigator.userAgent, subString: "OPR", identity: "Opera"},

            {string: navigator.userAgent, subString: "Chrome", identity: "Chrome"},
            {string: navigator.userAgent, subString: "Safari", identity: "Safari"}
        ]
    };
    BrowserDetect.init();
    const interval = setInterval(function() {
        $('.vimeorecorder').each(function(){
            var videoId = $(this).data('id');
            var pathname = window.location.pathname;
            var client = BrowserDetect.browser + ' ' + BrowserDetect.version;
            var identifier = $(this).attr('id');
            var iframe = document.querySelector('#' + identifier);
            var player = new Vimeo.Player(iframe);
            player.getCurrentTime().then(function(seconds) {
                $.ajax({
                    url: 'http://127.0.0.1:8000/api/v1/Watched',
                    type: 'POST',
                    contentType: "application/json; charset=utf-8",
                    dataType: "json",
                    data: JSON.stringify({
                        video: videoId,
                        identifier: 'aaaa',
                        url: pathname,
                        client: client,
                        watched: seconds,
                    }),
                });
            }).catch(function(error) {
                console.error( error );
            });
        });
    }, 5000);
</script>
```

You should modify the post url to that of your server (i.e. replace http://127.0.0.1:8000 in the URL field).
If you have a participant identifier change the identifier value to one appropriate for your project.
The 5000 is the time in ms between calls to the server, a call is made even if the video is NOT being watched, so you may wish to vary this.

In addition all of the vimeo videos that you are recordings stats for need to have the class:

```
class="vimeorecorder"
```

The identifiers for the videos are stored using data-id, e.g.

```
data-id="my-2"
```