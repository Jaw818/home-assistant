
# 🎉 Birthday Pop-up Card & Automations

![Birthday Card Preview](./preview.gif)

Welcome to the **Birthday Pop-up Card & Automations** project! 🎂🎈  
This was inspired by the incredibly creative [berkansezer77](https://github.com/berkansezer77) and borrows some of his brilliant components to bring birthdays to life on your Home Assistant dashboard.

It takes a little time to get everything set up, but once it's rolling, it’s an absolute treat. You _might_ find faster ways to do it, but this guide walks you through the process that worked best for me!

> ⚠️ First GitHub project here—please excuse any rookie formatting or explanation issues. I’m learning as I go!

---

## 📋 Project Overview

This guide shows you how to set up a fully automated and interactive Birthday Card experience on your Home Assistant dashboard. You'll also learn how to use calendar events as automation triggers for **push** and **TTS notifications**.

I started with birthdays, but I ended up expanding this setup for other calendars too—once you get going, the possibilities are endless!

---

## ✅ What You’ll Need

### Required:

- 📅 [Google Calendar Integration](http://home-assistant.io/integrations/google/)
- 📷 [Generic Camera](https://www.home-assistant.io/integrations/generic/)
- 🍄 [Mushroom Card](https://github.com/piitaya/lovelace-mushroom)

### Optional but Awesome:

- 🎨 [Card Mod](https://github.com/thomasloven/lovelace-card-mod)
- 🧱 [Vertical Stack In Card](https://github.com/ofekashery/vertical-stack-in-card)
- 🌐 [Browser Mod](https://github.com/thomasloven/hass-browser_mod)

> 📝 Note: My calendar events are named in the format “FirstName's Birthday.” This naming is essential for automations and photos later on. You can customize it, but this setup assumes that format.

---

## 🛠 Step 1: Set Up the Calendar Sensor

Let’s start by pulling upcoming events from your calendar.

Add this to your `configuration.yaml`, then update the calendar entity and reload from Developer Tools:

```yaml
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
          entity_id: calendar.YOUR_CALENDAR_HERE
        response_variable: raw_events
      - variables:
          scheduled_events: "{{ raw_events['calendar.YOUR_CALENDAR_HERE'] }}"
    sensor:
      - name: Calendar Scheduled Events
        unique_id: calendar_scheduled_events
        state: "{{ scheduled_events.events | count() }}"
        attributes:
          scheduled_events: "{{ scheduled_events.events }}"
        icon: mdi:calendar
```

This creates a new sensor: `sensor.calendar_scheduled_events`.

---

## 🧩 Step 2: Break Out the Data

Next, we extract useful info from that sensor.

You’ll create 4 new template sensors in **Settings > Devices & Services > Helpers > Template Sensor**.

### Example:

Name: `sensor.calendar_birthday_schedules_msg_1`  
Template:
```yaml
{{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[0].summary | replace("'s Birthday", "") }}
```

Repeat this process for sensors 2 through 4:

```yaml
{{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[1].summary | replace("'s Birthday", "") }}
{{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[2].summary | replace("'s Birthday", "") }}
{{ state_attr('sensor.calendar_scheduled_events', 'scheduled_events')[3].summary | replace("'s Birthday", "") }}
```

---

## 📅 Birthday Dates + Countdowns

You’ll also want to set up sensors to display the birthday date and how many days remain.

- [Birthday Date Sensor Templates](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Date%20Sensor%20Templates)
- [Days Until Birthday Sensor Templates](https://github.com/Jaw818/home-assistant/blob/main/Days%20until%20Birthday%20Sensor%20Templates)

---

## 🖼 Step 3: Dynamic Birthday Images

Here’s where it gets cool 😎

Instead of writing 70+ `if` conditions, I created dynamic image sensors that generate the file path based on the person’s name!

### Example Sensor:
```yaml
sensor.birthdayimg1
Template: /local/png/birthday/{{states('sensor.calendar_birthday_schedules_msg_1')}}.png
```

You’ll need 4 of these (just change the number). Then, upload matching images like `/local/png/birthday/Nicholas.png`.

---

## 📸 Step 4: Generic Cameras

Now display the images using 4 generic cameras:

**Camera URL:**
```
http://homeassistant:8123{{ states('sensor.birthdayimg1') }}
```

These will show up as `camera.birthday_1` through `camera.birthday_4`.

---

## 💬 Step 5: Optional Clean Output Sensors

These sensors give a cleaner message for the dashboard, showing things like “is Today” or the actual birthday date.

```yaml
{% if states('sensor.calendar_birthday_schedules_date_remain_1') == 'Far Away' %}
  {{ states.sensor.calendar_birthday_schedules_date_1.state }}
{% elif states('sensor.calendar_birthday_schedules_date_remain_1') == 'Tomorrow' %}
  is Tomorrow
{% elif states('sensor.calendar_birthday_schedules_date_remain_1') == 'Today' %}
  is Today
{% else %}
  is in {{ states('sensor.calendar_birthday_schedules_date_remain_1') }}
{% endif %}
```

---

## 🎨 The Cards

### Mobile Pop-up Version (Browser Mod):
<img src="https://github.com/user-attachments/assets/4287169e-ac3d-4c00-8311-143ac0ea55a4" alt="Alt Text" width="450" height="800">

📱 [Pop-up Birthday Card](https://github.com/Jaw818/home-assistant/blob/main/Popup%20Birthday%20Card)

### Wall Panel/Kiosk Version:
![Birthdays Redacted Kiosk](https://github.com/user-attachments/assets/55661e3e-fad3-40ac-bb00-8da8df973daf)
🖥 [Kiosk Birthday Card](https://github.com/Jaw818/home-assistant/blob/main/Kiosk%20Birthday%20Card)

> I used AI-generated children for my demo pictures…because, well, Photoshop fatigue is real.

---

## 🔔 Automations

### TTS Birthday Shoutouts:
This automation checks if someone’s birthday is today and announces it via your smart speakers.

🗣 [TTS Birthday Automation](https://github.com/Jaw818/home-assistant/blob/main/TTS%20Birthday%20Automation)

> I trigger mine when I turn off my morning alarm!

### Push Notifications:
![Screenshot_20250418_150353_One UI Home](https://github.com/user-attachments/assets/56c24cfb-0d95-4d4e-803d-84526fbfe451)

📲 [Day-of Push Notification](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Push%20Notifications)


🌅 [Night Before Reminder](https://github.com/Jaw818/home-assistant/blob/main/Night%20Before%20Birthday%20Reminder)

To make this work, set up a binary sensor like this:  
🔌 [Birthday Tomorrow Binary Sensor](https://github.com/Jaw818/home-assistant/blob/main/Birthday%20Binary%20Sensor)

---

## 🎁 Wrapping It Up

Thanks for checking out my birthday automation project! It started as a fun little idea and evolved into a fully dynamic dashboard experience.

If you try this out or make it better, I’d love to see your setup!  
Feel free to fork, star, or submit issues and improvements 🚀
