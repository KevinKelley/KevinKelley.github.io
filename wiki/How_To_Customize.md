The style sheet for this application can be changed by editing this page: [Default Style](/wiki/Default_Style.html) . Edit this file only if your are familiar with HTML cascading style sheets! Help with [http:colors.cgi colors]

The HTML template used for all pages can be switched to a different one using the [http:index.cgi?setup setup utility]. The file <tt>template.html</tt> is the default template. Templates can be edited using the [http:index.cgi?edittemplate edit template utility]. The file <tt>template.html</tt> should not be changed. This way it is always possible to go back to a working template. The template editor can be used to edit any HTML page in the wiki's main directory.

To change the title text of your Wiki edit [Title Text](/wiki/Title_Text.html) .

To change the title text which appears in the top of your browser frame [Browser Title](/wiki/Browser_Title.html) .

To change the copy right notice at the bottom edit [Copyright Text](/wiki/Copyright_Text.html) .

General content editing and formatting is explained in [How To Edit Text](/wiki/How_To_Edit_Text.html).

More customizing features can be found on the page [How To Use Advanced Features](/wiki/How_To_Use_Advanced_Features.html).

The following table shows the main layout of newLISP Wiki and the pages and styles which get loaded into the different areas.

```
[
|| <center> [Title Text](/wiki/Title_Text.html) and style .title</center> ||
||  [include: Home]  ||
|| <center> [Bottom Bar](/wiki/Bottom_Bar.html) and style .bottombar</center>||
|| <center> menu area (can be switched off using &#091;nomenu:&#093;)  style .menu</center> ||
|| <center> [Copyright Text](/wiki/Copyright_Text.html) and style .footer</center> ||
]
```
