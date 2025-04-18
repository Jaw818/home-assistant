# Birthday Pop-up Card & Automations
This project was heavily inspired, and uses some components of [berkansezer77's](https://github.com/berkansezer77 "berkansezer77") custom birthday card.

This project does require sometime to get setup, but once it is setup, it works great. (There _**may**_ be quicker ways to set this up, however, this write up will be the best way that I currently know.)

I apologize is this is hard to follow, read or just poorly formatted, this is my first project like this, and first project on Github.

#Project Description
This write up will walk you through the setup I used to create a Birthday Card for my Dashboard, as well as sensors that can easily be used as data and triggers for automations to send 
you push and tts notifications regarding calendar events. After setting this up for Birthdays, I also created one for a few of my other calendars for automation purposes.

#Requirements
**Calendar is required, I prefer Google Calendar, but you can adapt this to your needs**
- [Google Calendar](http://home-assistant.io/integrations/google/)
- [Generic Camera](https://www.home-assistant.io/integrations/generic/)
- [Mushroom Card](https://github.com/piitaya/lovelace-mushroom)

The following are only necessary if you would like to set it up identical to mine.
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)
- [Vertical Stack In Card](https://github.com/ofekashery/vertical-stack-in-card)
- [Browser Mod](https://github.com/thomasloven/hass-browser_mod)

#Note
In my birthday calendar, I named every event "first_name's Birthday" This is somewhat crucial for the pictures later in the project. You can likely adapt to fit your needs, however, for simplicity, I did it that way.
#Step 1: Configuring Foundational Calendar Sensors
This code was borrowed from the wonderful [berkansezer77](https://github.com/berkansezer77 "berkansezer77") if you haven't already, check out his projects, he has some pretty unique and pretty awesome projects.
This code needs to be placed in your configuration.yaml. After placing it there, and adding your calendar entities in, reload your configuration from developer tools.
``` yaml
template:
  - trigger:
      - platform: time_pattern
        minutes: /1
    action:
      - service: calendar.get_events
        data:
          duration:
            hours: 2191
            minutes: 0
            seconds: 0
          start_date_time: "{{ today_at() }}"
        target:
          entity_id: calendar.**your calendar entity here**
        response_variable: raw_events
      - variables:
          scheduled_events: "{{ raw_events['calendar.**your calendar entity here**'] }}"
    sensor:
      - name: Calendar Scheduled Events
        unique_id: calendar_scheduled_events
        state: "{{ scheduled_events.events | count() }}"
        attributes:
          scheduled_events: "{{ scheduled_events.events }}"
        icon: mdi:calendar'
```
After reloading, you will find that you have a new entity, called "sensor.calendar_scheduled_events".

#Step 2: Breaking out the Data
Still Courtsey of Berkansezer77, we will now be breaking out the data from sensor.calendar_scheduled_events.
We will be making a total of 12 sensors through the UI in this section. If you have never templated a sensor through the UI, navigate to the helpers section in the settings, then add helper, then template, and select sensor.

We will name the first sensor: "sensor.calendar_birthday_schedules_msg_1"

We will then use this as the template.
``` yaml {{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[0].summary | replace("'s Birthday", "") }}```

If you did it correctly, you should now see the name of the person with the next birthday in your calendar. 

The ```yaml | replace("'s Birthday","")``` porition will take the message from "Nicholas's Birthday" to just "Nicholas" which is vital for the photo part later in the project.

You will now need to create the same sensor, three additional times.

Name: sensor.calendar_birthday_schedules_msg_2

Template: ``` yaml {{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[1].summary | replace("'s Birthday", "") }}```

Name: sensor.calendar_birthday_schedules_msg_3

Template: ``` yaml {{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[2].summary | replace("'s Birthday", "") }} ```

Name: sensor.calendar_birthday_schedules_msg_4

Template: ``` yaml {{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[3].summary | replace("'s Birthday", "") }}```

Once you have created the 4 sensors, we will move onto the next. 

The next sensor is retreiving the date of the birthday on the calendar.
Create them the same way as above, with the following information.

[Birthday Date Sensor](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Date%20Sensor%20Templates)

For the final 4 sensors provided by Berkansezer77, which will deteremine the number of days until the birthday, use the following:

[Days Until Birthday Sensors](https://github.com/Jaw818/home-assistant/blob/main/Days%20until%20Birthday%20Sensor%20Templates)

#Still Sensors, but a little more fun
As I mentioned before, it was important to get the birthday person's name out seperately. I will now explain why.
After staring at my birthday calendar and realizing that there were over 70 birthdays on it, I was prepared to give up on Berkansezer77's birthday card, 
as they were using if statements to decide which photo to display, having over 70 birthdays, this would have been pretty rough to write, so I devised a different way to do it,
my way also prevents you from needing to change any yaml when you add a birthday to your calendar, only an image.

I created 4 more sensors, with this information:

Name: sensor.birthdayimg1

Template: ```yaml /local/png/birthday/{{states('sensor.calendar_birthday_schedules_msg_1')}}.png ```

Using this template, I am able to recieve back /local/png/birthday/Nicholas.png, which I have loaded into my local folder. But, it isn't exactly that simple.
After creating the 4 sensors we will move onto how we will show these photos.

#Generic Cameras
We will now create 4 generic camera entities with the following 
URL: http://homeassistant:8123{{states('sensor.birthdayimg1')}}
As long as you have images that match the sensor output, you should get it back.
I then named these cameras Birthday 1 - 4.

#Optional Sensors
I also created 4 of these sensors to provide cleaner outputs for dashboard purposes. Rather than returning 'Far Away' it will now return the actual date that the birthday is on.
Called sensor.birthdaystate1
```yaml
{%if states('sensor.calendar_birthday_schedules_date_remain_1') == 'Far Away' %}
{{states.sensor.calendar_birthday_schedules_date_1.state}}
{%elif states('sensor.calendar_birthday_schedules_date_remain_1') == 'Tomorrow' %}
is Tomorrow
{%elif states('sensor.calendar_birthday_schedules_date_remain_1') == 'Today' %}
is Today
{%else%}
is in {{states('sensor.calendar_birthday_schedules_date_remain_1')}}
{%endif%}
```
#The Cards
I replaced the photos of my friends and family with pictures of AI generated children. I got lazy when photoshopping, so the cards don't look perfect, I apologize.
First I have the Pop-up card, using browser mod. This card is intended for mobile devices mostly.
[Pop-up Birthday Card](https://github.com/Jaw818/home-assistant/blob/main/Popup%20Birthday%20Card)

This card is for wall panels and larger devices:

[Kiosk Birthday Card](https://github.com/Jaw818/home-assistant/blob/main/Kiosk%20Birthday%20Card)

#Automations
This first automation, uses a simple template to check to see if the birthday is occuring today, if it is, it will send a notification via TTS to your smart speakers.
I did not provide the entire automation, only the choose action, that way you can set your own triggers. I personally, have it trigger when I turn my morning alarm off. 
I also have the same automation setup to monitor each of the 4 birthdays, just incase 4 people have a birthday on the same day.
[TTS Birthday Announcement](https://github.com/Jaw818/home-assistant/blob/main/TTS%20Birthday%20Automation)

#Push Notifications:
These follow the same logic as the TTS notifications.
[Day of Birthday Notification](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Push%20Notifications)

This one requires the templating of a binary sensor. [Birthday Tomorrow Binary Sensor](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Binary%20Sensor)
[Night/Day Before Notification](https://github.com/Jaw818/home-assistant/blob/main/Night%20Before%20Birthday%20Reminder)
