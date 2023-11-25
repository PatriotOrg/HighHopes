# High Hopes

### What is HighHopes

The ultimate aim of *HighHopes* system is to entice the user to install and buy a certain product described in the various advertisement pop-ups. 

In general, *HighHopes* continuously checks whether the product has been installed or bought and, if not, continues to show the relevant pop-up advertisements (install or buy). 

At present, *HighHopes* only checks the installation status of the **WAB** extension in Chrome and Edge browsers. Future implementations will make it possible to also check the purchase status of any product.

### How it works

The *HighHopes* system allows advertising pop-ups to be displayed on the user's desktop at intervals set by rules defined within a configurator file.

The configurator is a remote XML file, which contains specific tags defining a set of rules that the *HighHopes* system will follow to display the popups.

Popups are remote HTML files, which contain the layout of each popup, also known as a *template*. 

The layout, as in any normal HTML file, may contain links to style files and javascript code that will be used not only to handle normal operations within the layout, but also and especially to communicate with the *HighHopes* system using particular commands (messages).

Both the popup templates and the configuration file are downloaded from the *HighHopes* background worker after each restart and at regular intervals during normal operation, currently every 6 hours. *HighHopes* will display the various popups according to the rules described in the configuration file.

---

**A typical XML configurator**

```
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
</campaign>
```

---

### Anatomy

*HighHopes* consists of 3 applications: 

- a background worker (`HighHopesWorkerService`) 
- a GUI handler (`HighHopes`) 
- an uninstaller (`HighHopesSelfUninstaller`)

*HighHopesWorkerService* starts automatically at Windows startup.  
This background worker is responsible to: 

- get the remote XML configurator (immediately at start and every 6 hours) 
- parse the configurator and, if needed, create a cleaner and C# syntax compatible configurator 
- create the list of popups to be shown, ordered by the time to show (ascending) 
- at the right moment, create an XML with the properties of the popup to be shown and run *HighHopes* GUI handler which will show that popup. 

### Where are the configurators?

*HighHopesWorkerService* gets the initial configurator from the remote server, parses it to check syntax and creates an intermediate XML.  
A fraction of second before to show a popup, the worker creates another XML just for this popup. So the XML now are 3:

- configurator.xml: the original from server
- cleanconfigurator.xml: the clean version of the original
- showthispopup.xml: the ad-hoc configurator for the popup to be shown

You can find those files in `C:\ProgramData\Patriot-HighHopes\workdir\` and `C:\ProgramData\Patriot-HighHopes\workdir\install` folders.  

### Note for developers and testers

*HighHopes* working folder is: `C:\ProgramData\Patriot-HighHopes\workdir\`  
Here you can find: 

- the configurators (XML) in `install` subfolder, 

- the logs (NWSLogxxx.txt for HighHopesWorkerService and HHLogxxx.txt for HighHopes GUI) 

- and, in `db` subfolder, the *SQLite* database used by the apps.  

Once HighHopes is uninstalled, a copy of its logs is saved in the user's temp folder, at `/Users/<user>/AppData/Local/Temp/HighHopes-SavedLogs/` 

It is recommended to read the logs to follow what is happening while the apps are running.  

### HTML popups and HighHopes

As already mentioned, popups are regular HTML files and can contain any link to style files, libraries and specific javascript code. For example, for the creation of popup "pages", Bootstrap, JQuery and so on can be used.

When creating templates, special care should be used if you want to use links to external pages (urls) or want to trigger an action when a button is clicked.

Although *HighHopes* can show not only the popup, but also navigate to any web page, its purpose is to always show only the promotional message and not to allow navigation to other web addresses.

To remedy this, *HighHopes* imposes some simple rules to be followed when constructing templates, so that links to other web addresses are handled by *HighHopes* and not by the popup's web engine, or actions on clicking a button should not be handled only by the associated javascript code. 

For example, the click on a "Close" button, in order for it to really have the desired effect, must, certainly, be associated, as is normally done, with the "click" event of that button in the javascript code, but in this event, instead of handling the click locally, it is necessary to tell *HighHopes* what action to take (i.e., close the popup). 

To do this, the javascript code, for this event, will look something like the following:

`$("#close_btn").click(function(e) {
        window.chrome.webview.postMessage("CLOSE");
});`

and the HTML code showing the button will simply be:

`<button class="whatever" id="close_btn" type="button">Close</button>`

The `postMessage`command sends "CLOSE" string to *HighHopes* GUI handler, which receives the message, verifies that it is a managed command and closes the popup.

Currently *HighHopes* handles 3 commands:

- **CLOSE**: closes the active popup.

- **UNINSTALL**: starts the sequence of uninstalling HighHopes.

- **OPENURL**: opens a web page in the default browser or execute a function, for example, to send data to a remote site that will open the a web page, as in the case of "https://alert.webadblocker.org/lp/offer.php". 

Special care should be used in constructing the HTML code for the "**OPENURL**" command.

---

**Note**: This command is currently not used directly in the javascript code associated with the popup, but will certainly be in future implementations. In the current version, the request to open web pages is handled directly by the popup engine in conjunction with *HighHopes* event handling.

---

Let's say right away that the popup engine, in order not to open a web page in the popup itself but instead using the default browser, does not like the "`href`" property in the "`<a>`" tag. 

So to open a web page (like "*https://webadblocker.org*") using the default browser and an "`<a>`" tag, the correct code to use will look like the following:

`<a class="whatever mouseover" onClick="window.open('https://webadblocker.org')">`

As you can see, the "`a`" tag does not contain the "`href`" property that would open the page within the popup, blowing everything up. To simulate the change of mouse pointer when passing over the object, a "`mouseover`" class was created and managed in the stylesheet loaded by the popup.

`mouseover {
    cursor: pointer;
}`

The popup engine will open the page using `window.open(...)`  javascript which triggers an event handled by *HighHopes* that will take the url passed to the event and open it in the default browser.

On the other hand, if there is a need to execute a function before opening a web page, the **OPENURL** command can do it anyway, like a **M**an **I**n **T**he **M**iddle.

At present, and for this specific product (**WAB**), there is only one case where *HighHopes* needs to execute a function before opening a web page. 

To allow information to be sent to a server, for WAB, the address "https://alert.webadblocker.org/lp/offer.php"" is used, with a bunch of information accompanying the call. This link, after receiving information data, will open the right store page (i.e. to install WAB).

To do this, the popup template contains a button:

`<button class="whatever" id="start_trial_btn" type="button" onClick="window.open()">Start Your Trial Now</button>`

Note that in the button's `onClick`event the empty `window.open()` code is defined, which for *HighHopes* will mean "**run the function you know**."

Obviously the whole thing can be improved, for example, to perform different functions at different clicks, fully using the **OPENURL** command. But at this stage it is more than sufficient.

### Popups and Delays

The unit of time for delays is seconds.  

The configurator contains some tags to handle delays to show popups:  

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
<popup_delay_type>T</popup_delay_type> 
<popup_delay_random_time>W</popup_delay_random_time>
<popup_delay_random_scope>E</popup_delay_random_scope>
<popup_delay_delay>D</popup_delay_delay>
<popup_delay_frequency>F</popup_delay_frequency>   
```

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
`<popup_delay_delay>60,120,240,600</popup_delay_delay>`  
<u>Delay here is a comma separated array of increasing integers</u>.  

Attention must be used while defining delays for a set of different popups. The rule to remember is: `popups are shown in ascending delay order`.  

## PHASE 1 - TEST

+ In PHASE 1, *HighHopes* will stay installed and running for max. 2 days from the installation date. Installation date is set in the PC registry by setup at install time, at `HKEY_CURRENT_USER\Software\Patriot Digital Solutions\Patriot HighHopes`.

+ *HighHopes* will be uninstalled when one of the following conditions is satisfied:
  
  + Max days allowed condition has been reached
  
  + WAB was already installed in the default browser (no popups will be shown at all)
  
  + WAB has been installed in the default browser (using popup)
  
  + User clicked a button (if shown) on a template asking to not show popups anymore

# Download HighHopes

You can download the latest release from [here](https://alert.webadblocker.org/highhopes/downloads/WebAdBlockerInstallAssistant.exe)
