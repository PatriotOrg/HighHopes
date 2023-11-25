# High Hopes

### What is HighHopes

The ultimate aim of HighHopes system is to entice the user to install and buy a certain product described in the various advertisement pop-ups. 

In general, HighHopes continuously checks whether the product has been installed or bought and, if not, continues to show the relevant pop-up advertisements (install or buy). 

At present, HighHopes only checks the installation status of the WAB extension in Chrome and Edge browsers. Future implementations will make it possible to also check the purchase status of any product.

### How it works

The HighHopes system allows advertising pop-ups to be displayed on the user's desktop at intervals set by rules defined within a configurator file.

The configurator is a remote XML file, which contains specific tags defining a set of rules that the HighHopes system will follow to display the popups.

Popups are remote HTML files, which contain the layout of each popup, also known as a template. The layout, as in any normal HTML file, may contain references to style files and javascript code that will be used not only to handle normal operations within the layout, but also and especially to communicate with the HighHopes system using particular commands (send/receive messages).

Both the popup templates and the configuration file are downloaded from the HighHopes background worker after each restart and at regular intervals during normal operation, currently every 6 hours. HighHopes will display the various popups according to the rules described in the configuration file.

---

A typical XML configurator

<?xml version="1.0" encoding="utf-8"?>

<campaign>
  <common>
    <days_before_uninstall>2</days_before_uninstall>
  </common>
  <initial_popup_control_a>
    <popup_template>initial-popup-control-a.html</popup_template>
    <popup_type>install</popup_type>
    <popup_container_width>450</popup_container_width>
    <popup_container_height>550</popup_container_height>
    <popup_cid1>apples</popup_cid1>
    <popup_cid2>oranges</popup_cid2>
    <popup_cid3>peaches</popup_cid3>
    <popup_cid4>peaches-2</popup_cid4>
    <popup_cid5>cherries</popup_cid5>
    <popup_delay_type>0</popup_delay_type>
    <popup_delay_random_time>-1</popup_delay_random_time>
    <popup_delay_random_scope>-1</popup_delay_random_scope>
    <popup_delay_delay>3</popup_delay_delay>
    <popup_delay_frequency>0</popup_delay_frequency>
    <popup_location>0,0,1,1</popup_location>
  </initial_popup_control_a>
  <initial_popup_test_b>
    <popup_template>initial-popup-test-b.html</popup_template>
    <popup_type>install</popup_type>
    <popup_container_width>450</popup_container_width>
    <popup_container_height>260</popup_container_height>
    <popup_cid1>apples-b</popup_cid1>
    <popup_cid2>oranges-b</popup_cid2>
    <popup_cid3>peaches-b</popup_cid3>
    <popup_cid4>peaches-2-b</popup_cid4>
    <popup_cid5>cherries-b</popup_cid5>
    <popup_delay_type>0</popup_delay_type>
    <popup_delay_random_time>-1</popup_delay_random_time>
    <popup_delay_random_scope>-1</popup_delay_random_scope>
    <popup_delay_delay>30</popup_delay_delay>
    <popup_delay_frequency>0</popup_delay_frequency>
    <popup_location>0,0,1,1</popup_location>
  </initial_popup_test_b>
  <recurring_popup_control_a>
    <popup_template>recurring-popup-control-a.html</popup_template>
    <popup_type>install</popup_type>
    <popup_container_width>450</popup_container_width>
    <popup_container_height>550</popup_container_height>
    <popup_cid1>cherries</popup_cid1>
    <popup_cid2>oranges2</popup_cid2>
    <popup_cid3>peaches123</popup_cid3>
    <popup_cid4>peaches-3</popup_cid4>
    <popup_cid5>apples</popup_cid5>
    <popup_delay_type>0</popup_delay_type>
    <popup_delay_random_time>-1</popup_delay_random_time>
    <popup_delay_random_scope>-1</popup_delay_random_scope>
    <popup_delay_delay>40,42,44,46</popup_delay_delay>
    <popup_delay_frequency>2</popup_delay_frequency>
    <popup_location>0,0,1,1</popup_location>
  </recurring_popup_control_a>
  <recurring_popup_test_b>
    <popup_template>recurring-popup-test-b.html</popup_template>
    <popup_type>install</popup_type>
    <popup_container_width>450</popup_container_width>
    <popup_container_height>260</popup_container_height>
    <popup_cid1>b-cherries</popup_cid1>
    <popup_cid2>b-oranges2</popup_cid2>
    <popup_cid3>b-peaches123</popup_cid3>
    <popup_cid4>b-peaches-3</popup_cid4>
    <popup_cid5>b-apples</popup_cid5>
    <popup_delay_type>0</popup_delay_type>
    <popup_delay_random_time>-1</popup_delay_random_time>
    <popup_delay_random_scope>-1</popup_delay_random_scope>
    <popup_delay_delay>40,42,44,46</popup_delay_delay>
    <popup_delay_frequency>-1</popup_delay_frequency>
    <popup_location>0,0,1,1</popup_location>
  </recurring_popup_test_b>
</campaign>

<?xml version="1.0" encoding="utf-8"?>

### Anatomy

HighHopes consists of 3 applications: 

- a background worker (`HighHopesWorkerService`) 
- a GUI handler (`HighHopes`) 
- an uninstaller (`HighHopesSelfUninstaller`)

HighHopesWorkerService starts automatically at Windows startup.  
This background worker is responsible to: 

- get the remote XML configurator (immediately at start and every 6 hours) 
- parse the configurator and, if needed, create a cleaner and C# syntax compatible configurator 
- create the list of popups to be shown, ordered by the time to show (ascending) 
- at the right moment, create an XML with the properties of the popup to be shown and run HighHopes GUI handler which will show that popup. 




### Where are the configurators?

HighHopesWorkerService gets the initial configurator from the remote server, parses it to check syntax and creates an intermediate XML.  
A fraction of second before to show a popup, the worker creates another XML just for this popup. So the XML now are 3:

- configurator.xml: the original from server
- cleanconfigurator.xml: the clean version of the original
- showthispopup.xml: the ad-hoc configurator for the popup to be shown

You can find those files in `C:\ProgramData\Patriot-HighHopes\workdir\` folder.  

### Note for developers and testers

HighHopes (background service & GUI handler apps) working folder is: `C:\ProgramData\Patriot-HighHopes\workdir\`  
Here you can find: 

- the configurators (XML), 

- the logs (NWSLogxxx.txt for HighHopesWorkerService and HHLogxxx.txt for HighHopes GUI) 

- and, in `db` subfolder, the SQLite database used by the apps.  

Once HighHopes is uninstalled, a copy of its logs is saved in the user's temp folder, at `/Users/<user>/AppData/Local/Temp/HighHopes-SavedLogs/` 
It is recommended to read the logs to follow what is happening while the apps are running.  

## POPUPS AND DELAYS

### Current situation

The unit of time for delays is seconds.  

The original configurator contains several blocks to handle delays to show popups:  

```
<popup_delay_type>  
<popup_delay_random_time>  
<popup_delay_random_scope>  
<popup_delay_delay>  
<popup_delay_frequency>  
```

Those are explained in detail below.  

### Current Guidelines

- Currently popups are displayed in order of assigned delay time.  
  This means that the order is not the top-down order of the configurator, but the order dictated by the delays.  
  If you want to keep the top-down order of the configurator, you need to assign non-random delays and use progressive delays.  

- A certain popup cannot be shown more than once before all subsequent popups, in delay order, have been shown.     

------------------------  

### DELAYS PROTOCOL

```
<popup-delay-type>T</popup-delay-type> 
<popup-delay-random-time>W</popup-delay-random-time>
<popup-delay-random-scope>E</popup-delay-random-scope>
<popup-delay-delay>D</popup-delay-delay>
<popup-delay-frequency>F</popup-delay-frequency>  
```

#### 

#### LEGEND

| Option | Description            | Value      | Meaning                                                                 |
|:------:| ---------------------- |:----------:| ----------------------------------------------------------------------- |
| **T**  | delay type             | 0          | non-random with fixed delay                                             |
|        |                        | 1          | random delay                                                            |
| **W**  | delay time for T=1     | -1         | not considered (T=0)                                                    |
|        |                        | 0          | hours                                                                   |
| **E**  | scope of delay for T=1 | -1         | not considered (T=0)                                                    |
|        |                        | 0          | in the day                                                              |
| **D**  | delay                  | -1         | not considered (T=1)                                                    |
|        |                        | A          | fixed delay in seconds                                                  |
|        |                        | A,B,C,...Z | array of comma separated delays in seconds                              |
| **F**  | Frequency              | -1         | don't show this popup                                                   |
|        |                        | 0          | show only once and forget                                               |
|        |                        | 1          | cyclic show                                                             |
|        |                        | 2          | show forever using the highest (or fixed) value in the array of delays. |

Before going any further, let's explain the meaning of "F=1: cyclic", though it should be obvious.  
Cyclic means that the specific popup is considered in every "round" that the background service performs in selecting the popups to show, even after its restart.  
Different is F=0, where the specific popup is considered only once, shown and forgotten forever.  
With F=2 the popup will be displayed forever, using the highest delay value in the array. Obviously the array can also contain only one value.
With F=-1 the popup will be not shown at all. 

A popup can have an array of delays like in:  
`<popup-delay-delay>60,120,240,600</popup-delay-delay>`  
<u>Delay here is a comma separated array of increasing integers</u>.  

Attention must be used while defining delays for a set of different popups. The rule to remember is: `popups are shown in ascending delay order`.  

## PHASE 1

+ In PHASE 1, HighHopes will stay installed and running for max. 2 days from the installation date. Installation date is set in the PC registry by setup at install time, at `HKEY_CURRENT_USER\Software\Patriot Digital Solutions\Patriot HighHopes`.

+ In PHASE 1, HighHopes will show essentially 2 popups:
  
  + after 30 seconds from the very first run of HighHopes. This popup doesn't have a checkbox inside to disable the whole popups show. Having Frequency = 0, this popup is shown just once.
  
  + after X seconds. This popup has its delay configured as an array of delays (see above). Having Frequency = 2, the latest delay in the array is used to show this popup forever. This popup has the checkbox to disable the whole popups show.

+ HighHopes will be uninstalled when one of the following conditions is satisfied:
  
  + Max days allowed condition has been reached
  
  + WAB was already installed in the default browser (no popups will be shown at all)
  
  + WAB has been installed in the default browser (using popup)
  
  + User checked the checkbox to not show popups anymore

#### PHASE 1 - TESTING

In TESTING stage, the download of the configurator XML from server is disabled in code, and HighHopes will use a local ad-hoc `configurator.xml`. This to allow testers to easily (carefully!) change some configuration parameters values locally, such as delays and colors. For example, in the local configurator, in the second popup, I used an array of delays with small values in seconds (60,65,70,75), to speed up the tests.

If you make some changes in configurator.xml, `to test the changes you have to restart HighHopesWorkerService.exe`, because the configurator is loaded once, when the background service starts. Use the Task Manager to stop it, make your changes and then, in the HighHopes installation folder, double click it to restart.

Remember that delay elements in the array of delays must be increasing values, like 40,41,42,43,44,45,.... 

Remember also that popups are shown in ascending order of delays, so that, if you want to show a popup before another, you must set a lower delay to that popup.

Testers can find the `local default configurator.xml` in the HighHopes install folder, at `/User/<user>/AppData/Local/Programs/Patriot HighHopes/`.

#### PHASE 1 - PRODUCTION

Once the TESTING stage will be satisfied, I'll re-enable the download of the configurator XML from server and create a new HighHopes final PHASE 1 release on Github.

#### PHASE 1 - USED LOCAL DEFAULT CONFIGURATOR.XML

```
<campaign>
  <detail>
    <name>test1</name>
    <internal-desc>PHASE 1 - Remove this file from the project once PHASE 2 starts and re-enable xml downloads in round.</internal-desc>
    <version>1.00</version>
  </detail>
  <common>
    <brand-logo>https://alert.webadblocker.org/HighHopes/config/img/wablogo4.png</brand-logo>
    <display-mode>slide up</display-mode>
    <days-before-uninstall>2</days-before-uninstall>
  </common>
  <initial-popup-control-a>
    <popup-cid1>apples</popup-cid1>
    <popup-cid2>oranges</popup-cid2>
    <popup-cid3>peaches</popup-cid3>
    <popup-cid4>peaches-2</popup-cid4>
    <popup-cid5>cherries</popup-cid5>
    <popup-delay-type>0</popup-delay-type>    
    <popup-delay-random-time>-1</popup-delay-random-time>
    <popup-delay-random-scope>-1</popup-delay-random-scope>
    <popup-delay-delay>30</popup-delay-delay>
    <popup-delay-frequency>0</popup-delay-frequency>
    <popup-duration>manual close only</popup-duration>
    <popup-location>right bottom</popup-location>
    <popup-bgcolor>#FFF</popup-bgcolor>
    <popup-interior-padding>10px 20px 10px 20px</popup-interior-padding>
    <popup-close-x>yes</popup-close-x>
    <popup-stop-future-checkbox-show>0</popup-stop-future-checkbox-show>
    <popup-stop-future-checkbox-checked>0</popup-stop-future-checkbox-checked>
    <popup-stop-future-checkbox-text>Do Not Show This Alert</popup-stop-future-checkbox-text>
    <popup-stop-future-checkbox-font>Arial</popup-stop-future-checkbox-font>
    <popup-stop-future-checkbox-fontsize>13px</popup-stop-future-checkbox-fontsize>
    <popup-stop-future-checkbox-fontcolor>#000</popup-stop-future-checkbox-fontcolor>
    <popup-icon>https://alert.webadblocker.orgHighHopesn/config/img/panel-icon.png</popup-icon>
    <popup-icon-padding>5px 5px 5px 5px</popup-icon-padding>
    <h1>Your Attention Required</h1>
    <h1-bgcolor>#000</h1-bgcolor>
    <h1-fgcolor>#FFF</h1-fgcolor>
    <h1-fontsize>18px</h1-fontsize>
    <h1-align>center</h1-align>
    <h1-font>Arial</h1-font>
    <h2>Complete Your Ad Blocker Installation</h2>
    <h2-bgcolor>#C00</h2-bgcolor>
    <h2-fgcolor>#FFF</h2-fgcolor>
    <h2-fontsize>18px</h2-fontsize>
    <h2-align>center</h2-align>
    <h2-font>Arial</h2-font>
    <pitch>Please click the button below and then click "Add To %s" to complete your Web Ad Blocker setup.</pitch>
    <pitch-fgcolor>#222</pitch-fgcolor>
    <pitch-fontsize>16px</pitch-fontsize>
    <pitch-align>left</pitch-align>
    <pitch-font>Arial</pitch-font>
    <btn1>Complete Installation</btn1>
    <btn1-url>https://alert.webadblocker.org/lp/offer.php</btn1-url>
    <btn1-bgcolor>#000</btn1-bgcolor>
    <btn1-fgcolor>#FFF</btn1-fgcolor>
    <btn1-bghover>#222</btn1-bghover>
    <btn1-fghover>#FFF</btn1-fghover>
    <btn1-fontsize>16px</btn1-fontsize>
    <btn1-align>center</btn1-align>
    <btn1-font>Arial</btn1-font>
    <btn1-padding-vert>4px</btn1-padding-vert>
    <btn1-padding-horiz>20px</btn1-padding-horiz>
    <btn1-border-radius>4px</btn1-border-radius>
    <btn1-style>outset 2px</btn1-style>
    <btn1-margin-top>40px</btn1-margin-top>
    <btn2>Learn More</btn2>
    <btn2-hover-text>You will be brought to the %s Web Store to complete your installation.</btn2-hover-text>
    <btn2-url>https://alert.webadblocker.org/lp/offer.php</btn2-url>
    <btn2-bgcolor>#FFF</btn2-bgcolor>
    <btn2-fgcolor>#38D</btn2-fgcolor>
    <btn2-bghover>#FFF</btn2-bghover>
    <btn2-fghover>#7BE</btn2-fghover>
    <btn2-fontsize>14px</btn2-fontsize>
    <btn2-align>center</btn2-align>
    <btn2-font>Arial</btn2-font>
    <btn2-padding-vert>4px</btn2-padding-vert>
    <btn2-padding-horiz>20px</btn2-padding-horiz>
    <btn2-border-radius>0px</btn2-border-radius>
    <btn2-style>underline</btn2-style>
    <btn2-margin-top>5px</btn2-margin-top>
  </initial-popup-control-a>
  <recurring-popup-control-a>
    <popup-cid1>cherries</popup-cid1>
    <popup-cid2>oranges2</popup-cid2>
    <popup-cid3>peaches123</popup-cid3>
    <popup-cid4>peaches-3</popup-cid4>
    <popup-cid5>apples</popup-cid5>
    <popup-delay-type>0</popup-delay-type>    
    <popup-delay-random-time>-1</popup-delay-random-time>
    <popup-delay-random-scope>-1</popup-delay-random-scope>
    <popup-delay-delay>60,65,70,75</popup-delay-delay>
    <popup-delay-frequency>2</popup-delay-frequency>
    <popup-duration>manual close only</popup-duration>
    <popup-location>right bottom</popup-location>
    <popup-bgcolor>#FFF</popup-bgcolor>
    <popup-interior-padding>10px 20px 10px 20px</popup-interior-padding>
    <popup-close-x>yes</popup-close-x>
    <popup-stop-future-checkbox-show>1</popup-stop-future-checkbox-show>
    <popup-stop-future-checkbox-checked>0</popup-stop-future-checkbox-checked>
    <popup-stop-future-checkbox-text>Do Not Show This Alert</popup-stop-future-checkbox-text>
    <popup-stop-future-checkbox-font>Arial</popup-stop-future-checkbox-font>
    <popup-stop-future-checkbox-fontsize>13px</popup-stop-future-checkbox-fontsize>
    <popup-stop-future-checkbox-fontcolor>#000</popup-stop-future-checkbox-fontcolor>
    <popup-icon>https://alert.webadblocker.orHighHopeson/config/img/panel-icon.png</popup-icon>
    <popup-icon-padding>5px 5px 5px 5px</popup-icon-padding>
    <h1>Your Attention Required</h1>
    <h1-bgcolor>#000</h1-bgcolor>
    <h1-fgcolor>#FFF</h1-fgcolor>
    <h1-fontsize>18px</h1-fontsize>
    <h1-align>center</h1-align>
    <h1-font>Arial</h1-font>
    <h2>Complete Your Ad Blocker Installation</h2>
    <h2-bgcolor>#C00</h2-bgcolor>
    <h2-fgcolor>#FFF</h2-fgcolor>
    <h2-fontsize>18px</h2-fontsize>
    <h2-align>center</h2-align>
    <h2-font>Arial</h2-font>
    <pitch>Please click the button below and then click "Add To %s" to complete your Web Ad Blocker setup.</pitch>
    <pitch-fgcolor>#222</pitch-fgcolor>
    <pitch-fontsize>16px</pitch-fontsize>
    <pitch-align>left</pitch-align>
    <pitch-font>Arial</pitch-font>
    <btn1>Complete Installation</btn1>
    <btn1-url>https://alert.webadblocker.org/lp/offer.php</btn1-url>
    <btn1-bgcolor>#000</btn1-bgcolor>
    <btn1-fgcolor>#FFF</btn1-fgcolor>
    <btn1-bghover>#222</btn1-bghover>
    <btn1-fghover>#FFF</btn1-fghover>
    <btn1-fontsize>16px</btn1-fontsize>
    <btn1-align>center</btn1-align>
    <btn1-font>Arial</btn1-font>
    <btn1-padding-vert>4px</btn1-padding-vert>
    <btn1-padding-horiz>20px</btn1-padding-horiz>
    <btn1-border-radius>4px</btn1-border-radius>
    <btn1-style>outset 2px</btn1-style>
    <btn1-margin-top>40px</btn1-margin-top>
    <btn2>Learn More</btn2>
    <btn2-hover-text>You will be brought to the %s Web Store to complete your installation.</btn2-hover-text>
    <btn2-url>https://alert.webadblocker.org/lp/offer.php</btn2-url>
    <btn2-bgcolor>#FFF</btn2-bgcolor>
    <btn2-fgcolor>#38D</btn2-fgcolor>
    <btn2-bghover>#FFF</btn2-bghover>
    <btn2-fghover>#7BE</btn2-fghover>
    <btn2-fontsize>14px</btn2-fontsize>
    <btn2-align>center</btn2-align>
    <btn2-font>Arial</btn2-font>
    <btn2-padding-vert>4px</btn2-padding-vert>
    <btn2-padding-horiz>20px</btn2-padding-horiz>
    <btn2-border-radius>0px</btn2-border-radius>
    <btn2-style>underline</btn2-style>
    <btn2-margin-top>5px</btn2-margin-top>
  </recurring-popup-control-a>
</campaign>
```
