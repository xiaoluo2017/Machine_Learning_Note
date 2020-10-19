# DateTime Programming Guideline
* This page will describe some detailed knowledge of DateTime Programming best practice, including including using different DateTime storage methods for past, present and future events, how to create or convert datetime to a specific time zone, how to check whether the dateTime entered by the user are ambiguous or skipped times, and some corner cases involving leap years.

## Basic Questions
### What is special about DateTime objects？
* A DateTime object can fall into one of three different categories, the DateTime object has a property called "Kind" that represents each one of the possible categories: UTC, Unspecified and Local
### What is different between UTC, Unspecified and Local DateTime?
* UTC: Coordinated Universal Time (or UTC) is the primary time standard by which the world regulates clocks and time. It is precise and unambiguous.
* Local: A local DateTime value only knows that it’s local in relation to the machine where it was generated.(Remember to always convert dates to UTC before performing arithmetic with them. DateTime.Now is OK to use when you just want to display the current time on your application’s screen)
* Undefined: When saving dateTime to the database, the type attribute information will be lost. UTC/local will change to unspecified.
### When should we use UTC？
* One sentence summary, Use UTC for past past and present events, not use Utc for future local events
* Consider a situation where the user sets an alarm clock before going to bed at 8 o'clock tomorrow morning. If tonight, the summer time/winter time conversion will occur. If we use UTC to record the time, the next morning, when we calculate through the recorded UTC time and time zone, the obtained alarm time will become 7 or 9 o'clock, which is not what the user wants. This is why we use UTC in past and present events, because the UTC of past and present events is accurate.
* Use Local time(or DateTimeOffset) and TimeZoneInfo for future events, which will be detailed explained in section 2.
### What is different between DateTime.Now and DateTime.UTCNow？
* DateTime.Now return current local time, DateTime.UTCNow return current UTC time. 
* Use DateTime.UtcNow to store dates for past events and always use them for calculations, convert to local time when displaying the current time on your application’s screen
* DateTime.Now is also not unit-testing friendly. If some portion of your code includes a call to DateTime.Now, it means that it depends on an external data source it doesn't control.

## 1. Use UTC for past events
* Use DateTime.UtcNow to get present UTC time
```
DateTime now = DateTime.UtcNow;
```

* Convert to UTC time for calculations
```
var start = DateTime.Now; // local time
var end = DateTime.Now; // local time
var duration = end.ToUniversalTime() - start.ToUniversalTime(); // converting to UTC
```

> ref: https://blog.submain.com/4-common-datetime-mistakes-c-avoid/

## 2. Not use UTC for future local events
* Note: When saving dateTime to the database, the type attribute information will be lost (UTC/local->unspecified). Either explicitly indicate whether this is UTC time or local time on the column, or use DateTimeOffset

* Solution 1, implicit:
    * Save DateTime using Local time and TimeZoneInfo for future events
    * When you store UTC time or local time in the database, My suggestion here is to make it explicit that you’re doing so. Take UTC time as an example, append the "UTC" suffix to every database column and class property that holds a UTC datetime. Instead of Created, change it to CreatedUTC and so on. So does local time.
```
// save DateTime using Local time and TimeZoneInfo
public class Conference
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
    public LocalDateTime LocalStart { get; set; } // Local start: date/time in the specified time zone
    public string TimeZoneId { get; set; } // Time zone ID: string
    public Instant UtcStart { get; set; }
    public string TimeZoneRules { get; set; } // UTC start: derived field for convenience
}

// In other code... some parameters might be fields in the class.
public void UpdateUtcStart(
    Conference conference,
    IDateTimeZoneProvider latestZoneProvider)
{
    DateTimeZone newZone = latestZoneProvider[conference.TimeZoneId];
    // Preserve the local time, but with the new time zone rules
    ZonedDateTime newZonedStart = conference.LocalStart.InZoneLeniently(newZone);
 
    // Update the conference entry with the new information
    conference.UtcStart = newZonedStart.ToInstant();
    conference.TimeZoneRules = latestZoneProvider.VersionId;
}
```
* Solution 2, explicit:
    * Save DateTime using DateTimeOffset and TimeZoneInfo for future events
```
// save DateTime using DateTimeOffset and TimeZoneInfo
// Start: 2022-07-10T09:00:00+02:00
// TimeZoneId: Europe/Amsterdam
```

> ref: https://codeblog.jonskeet.uk/2019/03/27/storing-utc-is-not-a-silver-bullet/
> ref: https://stackoverflow.com/questions/4331189/datetime-vs-datetimeoffset
> ref: https://docs.microsoft.com/en-us/dotnet/api/system.datetimeoffset?view=netframework-4.7.2

## 3. Check whether the dateTime entered by the user are ambiguous or skipped times
* When the summer time/winter time is converted, for the local time of the converted time zone, there will be 1 hour that disappears or 1 hour that repeats twice. When the user sets these times, there will be ambiguous or skip time errors. We Need to catch these errors for processing
```
TimeZoneInfo tz = TimeZoneInfo.Local; // getting the current system timezone
DateTime dateTime = GetDateTimeFromUserInput(); // or another external untrusted source

if (tz.IsAmbiguousTime(dateTime))
{
    // do something
}

if (tz.IsInvalidTime(dateTime))
{
    // do something
}

// seems good to go!
```

## 4. Create datetime in a specific time zone
* The following requirements are not commonly used, but if you need to obtain the time in a specific time zone, or convert the current time to the time in a specific time zone, you can use the following template.
* Note: For specific timeZone Id List, you can see the following answers: https://stackoverflow.com/questions/14149346/what-value-should-i-pass-into-timezoneinfo-findsystemtimezonebyidstring/24460750#24460750
```
public struct DateTimeWithZone
{
    private readonly DateTime utcDateTime;
    private readonly TimeZoneInfo timeZone;

    public DateTimeWithZone(DateTime dateTime, TimeZoneInfo timeZone)
    {
        var dateTimeUnspec = DateTime.SpecifyKind(dateTime, DateTimeKind.Unspecified);
        utcDateTime = TimeZoneInfo.ConvertTimeToUtc(dateTimeUnspec, timeZone); 
        this.timeZone = timeZone;
    }

    public DateTime UniversalTime { get { return utcDateTime; } }

    public TimeZoneInfo TimeZone { get { return timeZone; } }

    public DateTime LocalTime
    { 
        get 
        { 
            return TimeZoneInfo.ConvertTime(utcDateTime, timeZone); 
        }
    }        
}
```

```
public static DateTime GmtToPacific(DateTime dateTime)
{
    return TimeZoneInfo.ConvertTimeFromUtc(dateTime, TimeZoneInfo.FindSystemTimeZoneById("Pacific Standard Time"));
		
    // Another way
    // return TimeZoneInfo.ConvertTimeBySystemTimeZoneId(DateTime.UtcNow, "Central Standard Time");
}
```

```
// DateTimeOffset is not associated with any time zone, it cannot possibly apply DST rules, DST is a rule of a particular time zone.
// You need also record TimeZoneInfo.
DateTimeOffset do1 = new DateTimeOffset(2008, 8, 22, 1, 0, 0, new TimeSpan(-5, 0, 0));
```
> ref: https://stackoverflow.com/questions/246498/creating-a-datetime-in-a-specific-time-zone-in-c-sharp

## 5. Leap year related
* For all problems related to leap years, we can use system functions to avoid

### 5.1 Check if the year is leap year
* DateTime.IsLeapYear()
```
int year = 2019; // year must be between 1 and 9999
bool isLeapYear = DateTime.IsLeapYear(year); // isLeapYear == false
```
> ref: https://www.tutorialspoint.com/datetime-isleapyear-method-in-chash


### 5.2 Calculate the current day-of-year
* There's a DateTime property named: DayOfYear
```
Console.WriteLine(DateTime.Now.DayOfYear);
```

* Or for any date:
```
var d = new DateTime(2010, 5, 30);
Console.WriteLine(d.DayOfYear);
```
> ref: https://stackoverflow.com/questions/2923193/c-sharp-how-to-calculate-the-current-day-of-year

### 5.3 Getting number of days in a month
```
int days = DateTime.DaysInMonth(year, month);
```
