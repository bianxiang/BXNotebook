只提供日期选择，不提供时间选择！

[toc]

http://api.jqueryui.com/datepicker/

Datepicker高度可配置。可以定制日期格式、语言。限制可选的日期范围。

By default, the datepicker calendar opens in a small overlay when the associated text field gains focus. For an inline calendar, simply attach the datepicker to a div or span.

This widget manipulates its element's value programmatically, therefore a native change event may not be fired when the element's value changes.

Creating a datepicker on an `<input type="date">` is **not** supported due to a UI conflict with the native picker.

### 键盘快捷键

当datepicker打开时，下面的命令是可用：

- `PAGE UP`：上一个月
- `PAGE DOWN`：下一个月
- `CTRL + PAGE UP`：上一年
- `CTRL + PAGE DOWN`：下一年
- `CTRL + HOME`：打开关闭的datepicker
- `CTRL/COMMAND + HOME`：移到当前月。
- `CTRL/COMMAND + LEFT`: Move to the previous day.
- `CTRL/COMMAND + RIGHT`: Move to the next day.
- `CTRL/COMMAND + UP`: Move to the previous week.
- `CTRL/COMMAND + DOWN`: Move the next week.
- `ENTER`: Select the focused date.
- `CTRL/COMMAND + END`: Close the datepicker and erase the date.
- `ESCAPE`: Close the datepicker without selection.

### 工具函数

#### 全局设置setDefaults

	$.datepicker.setDefaults( settings )

修改全局默认设置。

> Use the `option()` method to change settings for individual instances.

例子：

Set all date pickers to open on focus or a click on an icon.

    $.datepicker.setDefaults({
      showOn: "both",
      buttonImageOnly: true,
      buttonImage: "calendar.gif",
      buttonText: "Calendar"
    });

Set all date pickers to have French text.

	$.datepicker.setDefaults( $.datepicker.regional[ "fr" ] );

#### 格式化日期formatDate

	$.datepicker.formatDate( format, date, settings )

Format a date into a string value with a specified format.

The format can be combinations of the following:

- `d` - day of month (no leading zero)
- `dd` - day of month (two digit)
- `o` - day of the year (no leading zeros)
- `oo` - day of the year (three digit)
- `D` - day name short
- `DD` - day name long
- `m` - month of year (no leading zero)
- `mm` - month of year (two digit)
- `M` - month name short
- `MM` - month name long
- `y` - year (two digit)
- `yy` - year (four digit)
- `@` - Unix timestamp (ms since 01/01/1970)
- `!` - Windows ticks (100ns since 01/01/0001)
- `'...'` - literal text
- `''` - single quote
- anything else - literal text

There are also a number of predefined standard date formats available from $.datepicker:

- `ATOM` - 'yy-mm-dd' (Same as RFC 3339/ISO 8601)
- COOKIE - 'D, dd M yy'
- ISO_8601 - 'yy-mm-dd'
- RFC_822 - 'D, d M y' (See RFC 822)
- RFC_850 - 'DD, dd-M-y' (See RFC 850)
- RFC_1036 - 'D, d M y' (See RFC 1036)
- RFC_1123 - 'D, d M yy' (See RFC 1123)
- RFC_2822 - 'D, d M yy' (See RFC 2822)
- RSS - 'D, d M y' (Same as RFC 822)
- TICKS - '!'
- `TIMESTAMP` - '@'
- `W3C` - 'yy-mm-dd' (Same as ISO 8601)

例子：

	$.datepicker.formatDate( "yy-mm-dd", new Date( 2007, 1 - 1, 26 ) );

Display the date in expanded French format. Produces "Samedi, Juillet 14, 2007".

    $.datepicker.formatDate( "DD, MM d, yy", new Date( 2007, 7 - 1, 14 ), {
      dayNamesShort: $.datepicker.regional[ "fr" ].dayNamesShort,
      dayNames: $.datepicker.regional[ "fr" ].dayNames,
      monthNamesShort: $.datepicker.regional[ "fr" ].monthNamesShort,
      monthNames: $.datepicker.regional[ "fr" ].monthNames
    });

#### 解析日期parseDate

	$.datepicker.parseDate( format, value, settings )

Extract a date from a string value with a specified format.

The format can be combinations of the following:

- `d` - day of month (no leading zero)
- `dd` - day of month (two digit)
- `o` - day of the year (no leading zeros)
- `oo` - day of the year (three digit)
- `D` - day name short
- `DD` - day name long
- `m` - month of year (no leading zero)
- `mm` - month of year (two digit)
- `M` - month name short
- `MM` - month name long
- `y` - year (two digit)
- `yy` - year (four digit)
- `@` - Unix timestamp (ms since 01/01/1970)
- `!` - Windows ticks (100ns since 01/01/0001)
- `'...'` - literal text
- `''` - single quote
- anything else - literal text

A number of exceptions may be thrown:

- 'Invalid arguments' if either format or value is null
- 'Missing number at position nn' if format indicated a numeric value that is not then found
- 'Unknown name at position nn' if format indicated day or month name that is not then found
- 'Unexpected literal at position nn' if format indicated a literal value that is not then found
- 'Invalid date' if the date is invalid, such as '31/02/2007'

例子：

	$.datepicker.parseDate( "yy-mm-dd", "2007-01-26" );

Extract a date in expanded French format.

    $.datepicker.parseDate( "DD, MM d, yy", "Samedi, Juillet 14, 2007", {
      shortYearCuroff: 20,
      dayNamesShort: $.datepicker.regional[ "fr" ].dayNamesShort,
      dayNames: $.datepicker.regional[ "fr" ].dayNames,
      monthNamesShort: $.datepicker.regional[ "fr" ].monthNamesShort,
      monthNames: $.datepicker.regional[ "fr" ].monthNames
    });

#### 获取日期中的周iso8601Week

	$.datepicker.iso8601Week( date )

Determine the week of the year for a given date: 1 to 53.

This function uses the ISO 8601 definition of a week: weeks start on a Monday and the first week of the year contains January 4. This means that up to three days from the previous year may be included in the of first week of the current year, and that up to three days from the current year may be included in the last week of the previous year.

This function is the default implementation for the `calculateWeek` option.

	$.datepicker.iso8601Week( new Date( 2007, 1 - 1, 26 ) );

#### noWeekends

	$.datepicker.noWeekends

Set as `beforeShowDay` function to prevent selection of weekends.

We can provide the `noWeekends()` function into the `beforeShowDay` option which will calculate all the weekdays and provide an array of true/false values indicating whether a date is selectable.

Code examples:
Set the DatePicker so no weekend is selectable

    $( "#datepicker" ).datepicker({
      beforeShowDay: $.datepicker.noWeekends
    });

### Localization

Datepicker provides support for localizing its content to cater for different languages and date formats. Each localization is contained within its own file with the language code appended to the name, e.g., `jquery.ui.datepicker-fr.js` for French. The desired localization file should be included after the main datepicker code. Each localization file adds its settings to the set of available localizations and automatically applies them as defaults for all instances. Localization files can be found at **https://github.com/jquery/jquery-ui/tree/master/ui/i18n**.

The `$.datepicker.regional` attribute holds an array of localizations, indexed by language code, with "" referring to the default (English). Each entry is an object with the following attributes: closeText, prevText, nextText, currentText, monthNames, monthNamesShort, dayNames, dayNamesShort, dayNamesMin, weekHeader, dateFormat, firstDay, isRTL, showMonthAfterYear, and yearSuffix.

You can restore the default localizations with:

	$.datepicker.setDefaults( $.datepicker.regional[ "" ] );

And can then override an individual datepicker for a specific locale:

	$( selector ).datepicker( $.datepicker.regional[ "fr" ] );

### （未）Theming

### 选项

#### altField

Type: Selector or jQuery or Element
Default: ""

An input element that is to be updated with the selected date from the datepicker. Use the altFormat option to change the format of the date within this field. Leave as blank for no alternate field.

Code examples:

Initialize the datepicker with the altField option specified:

    $( ".selector" ).datepicker({
      altField: "#actualDate"
    });

Get or set the altField option, after initialization:

    // Getter
    var altField = $( ".selector" ).datepicker( "option", "altField" );
    // Setter
    $( ".selector" ).datepicker( "option", "altField", "#actualDate" );

#### altFormat

Type: String
Default: ""

The `dateFormat` to be used for the `altField` option. This allows one date format to be shown to the user for selection purposes, while a different format is actually sent behind the scenes. For a full list of the possible formats see the `formatDate` function

Initialize the datepicker with the `altFormat` option specified:

    $( ".selector" ).datepicker({
      altFormat: "yy-mm-dd"
    });

Get or set the altFormat option, after initialization:

    // Getter
    var altFormat = $( ".selector" ).datepicker( "option", "altFormat" );
    // Setter
    $( ".selector" ).datepicker( "option", "altFormat", "yy-mm-dd" );

#### appendText

Type: String
Default: ""

The text to display after each date field, 例如，显示正确的输入格式。

Code examples:

Initialize the datepicker with the appendText option specified:

    $( ".selector" ).datepicker({
      appendText: "(yyyy-mm-dd)"
    });

Get or set the appendText option, after initialization:

    // Getter
    var appendText = $( ".selector" ).datepicker( "option", "appendText" );
    // Setter
    $( ".selector" ).datepicker( "option", "appendText", "(yyyy-mm-dd)" );

#### autoSize

Type: Boolean
Default: false

Set to true to automatically resize the input field to accommodate dates in the current dateFormat.

Code examples:
Initialize the datepicker with the autoSize option specified:

    $( ".selector" ).datepicker({
      autoSize: true
    });

Get or set the autoSize option, after initialization:

    // Getter
    var autoSize = $( ".selector" ).datepicker( "option", "autoSize" );
    // Setter
    $( ".selector" ).datepicker( "option", "autoSize", true );

#### beforeShow

Type: Function( Element input, Object inst )
Default: null

A function that takes an input field and current datepicker instance and returns an options object to update the datepicker with. It is called just before the datepicker is displayed.

#### beforeShowDay

Type: Function( Date date )
Default: null

A function that takes a date as a parameter and must return an array with:

- [0]: true/false indicating whether or not this date is selectable
- [1]: a CSS class name to add to the date's cell or "" for the default presentation
- [2]: an optional popup tooltip for this date
The function is called for each day in the datepicker before it is displayed.

#### buttonImage

Type: String
Default: ""

A URL of an image to use to display the datepicker when the `showOn` option is set to "button" or "both". If set, the buttonText option becomes the alt value and is not directly displayed.

Code examples:

Initialize the datepicker with the buttonImage option specified:

    $( ".selector" ).datepicker({
      buttonImage: "/images/datepicker.gif"
    });

Get or set the buttonImage option, after initialization:

    // Getter
    var buttonImage = $( ".selector" ).datepicker( "option", "buttonImage" );
    // Setter
    $( ".selector" ).datepicker( "option", "buttonImage", "/images/datepicker.gif" );

#### buttonImageOnly

Type: Boolean
Default: false

Whether the button image should be rendered by itself instead of inside a button element. This option is only relevant if the `buttonImage` option has also been set.


    $( ".selector" ).datepicker({
      buttonImageOnly: true
    });

#### buttonText

Type: String
Default: "..."

The text to display on the trigger button. Use in conjunction with the `showOn` option set to "button" or "both".

    $( ".selector" ).datepicker({
      buttonText: "Choose"
    });

#### calculateWeek

Type: Function()
Default: jQuery.datepicker.iso8601Week

A function to calculate the week of the year for a given date. The default implementation uses the ISO 8601 definition: weeks start on a Monday; the first week of the year contains the first Thursday of the year.

    $( ".selector" ).datepicker({
      calculateWeek: myWeekCalc
    });


#### changeMonth

Type: Boolean
Default: false

Whether the month should be rendered as a dropdown instead of text.

    $( ".selector" ).datepicker({
      changeMonth: true
    });

#### changeYear

Type: Boolean
Default: false

Whether the year should be rendered as a dropdown instead of text. Use the `yearRange` option to control which years are made available for selection.

    $( ".selector" ).datepicker({
      changeYear: true
    });

#### closeText

Type: String
Default: "Done"

The text to display for the close link. Use the `showButtonPanel` option to display this button.

    $( ".selector" ).datepicker({
      closeText: "Close"
    });

#### constrainInput

Type: Boolean
Default: true

When true, entry in the input field is constrained to those characters allowed by the current `dateFormat` option.

    $( ".selector" ).datepicker({
      constrainInput: false
    });

#### currentText

Type: String
Default: "Today"

The text to display for the current day link. Use the `showButtonPanel` option to display this button.

    $( ".selector" ).datepicker({
      currentText: "Now"
    });

#### dateFormat

Type: String
Default: "mm/dd/yy"

The format for parsed and displayed dates. For a full list of the possible formats see the `formatDate` function.

    $( ".selector" ).datepicker({
      dateFormat: "yy-mm-dd"
    });

#### dayNames

Type: Array
Default: [ "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" ]

The list of long day names, starting from Sunday, for use as requested via the `dateFormat` option.

    $( ".selector" ).datepicker({
      dayNames: [ "Dimanche", "Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi", "Samedi" ]
    });

#### dayNamesMin

Type: Array
Default: [ "Su", "Mo", "Tu", "We", "Th", "Fr", "Sa" ]

The list of minimised day names, starting from Sunday, for use as column headers within the datepicker.

    $( ".selector" ).datepicker({
      dayNamesMin: [ "Di", "Lu", "Ma", "Me", "Je", "Ve", "Sa" ]
    });

#### dayNamesShort

Type: Array
Default: [ "Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat" ]

The list of abbreviated day names, starting from Sunday, for use as requested via the dateFormat option.

    $( ".selector" ).datepicker({
      dayNamesShort: [ "Dim", "Lun", "Mar", "Mer", "Jeu", "Ven", "Sam" ]
    });

#### defaultDate

Type: Date or Number or String
Default: null

Set the date to highlight on first opening if the field is blank. Specify either an actual date via a `Date` object or as a string in the current dateFormat, or a number of days from today (e.g. `+7`) or a string of values and periods ('y' for years, 'm' for months, 'w' for weeks, 'd' for days, e.g. '+1m +7d'), or null for today.

Multiple types supported:

- Date: A date object containing the default date.
- Number: A number of days from today. For example 2 represents two days from today and -1 represents yesterday.
- String: A string in the format defined by the dateFormat option, or a relative date. Relative dates must contain value and period pairs; valid periods are "y" for years, "m" for months, "w" for weeks, and "d" for days. For example, "+1m +7d" represents one month and seven days from today.

例子：

    $( ".selector" ).datepicker({
      defaultDate: +7
    });

#### duration

Type: or String
Default: "normal"

Control the speed at which the datepicker appears, it may be a time in milliseconds or a string representing one of the three predefined speeds ("slow", "normal", "fast").

    $( ".selector" ).datepicker({
      duration: "slow"
    });

#### firstDay

Type: Integer
Default: 0

Set the first day of the week: Sunday is 0, Monday is 1, etc.

    $( ".selector" ).datepicker({
      firstDay: 1
    });

#### gotoCurrent

Type: Boolean
Default: false

When true, the current day link moves to the currently selected date instead of today.

    $( ".selector" ).datepicker({
      gotoCurrent: true
    });

#### hideIfNoPrevNext

Type: Boolean
Default: false

Normally the previous and next links are disabled when not applicable (see the `minDate` and `maxDate` options). You can hide them altogether by setting this attribute to true.

    $( ".selector" ).datepicker({
      hideIfNoPrevNext: true
    });

#### isRTL

Type: Boolean
Default: false

Whether the current language is drawn from right to left.

    $( ".selector" ).datepicker({
      isRTL: true
    });

#### maxDate

Type: Date or Number or String
Default: null

The maximum selectable date. When set to null, there is no maximum.

Multiple types supported:

- Date: A date object containing the maximum date.
- Number: A number of days from today. For example 2 represents two days from today and -1 represents yesterday.
- String: A string in the format defined by the dateFormat option, or a relative date. Relative dates must contain value and period pairs; valid periods are "y" for years, "m" for months, "w" for weeks, and "d" for days. For example, "+1m +7d" represents one month and seven days from today.

    $( ".selector" ).datepicker({
      maxDate: "+1m +1w"
    });

#### minDate

Type: Date or Number or String
Default: null

The minimum selectable date. When set to null, there is no minimum.

Multiple types supported:

- Date: A date object containing the minimum date.
- Number: A number of days from today. For example 2 represents two days from today and -1 represents yesterday.
- String: A string in the format defined by the dateFormat option, or a relative date. Relative dates must contain value and period pairs; valid periods are "y" for years, "m" for months, "w" for weeks, and "d" for days. For example, "+1m +7d" represents one month and seven days from today.


    $( ".selector" ).datepicker({
      minDate: new Date(2007, 1 - 1, 1)
    });

#### monthNames

Type: Array
Default: [ "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December" ]

The list of full month names, for use as requested via the dateFormat option.

    $( ".selector" ).datepicker({
      monthNames: [ "Januar", "Februar", "Marts", "April", "Maj", "Juni", "Juli", "August", "September", "Oktober", "November", "December" ]
    });

#### monthNamesShort

Type: Array
Default: [ "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec" ]

The list of abbreviated month names, as used in the month header on each datepicker and as requested via the `dateFormat` option.

    $( ".selector" ).datepicker({
      monthNamesShort: [ "Jan", "Feb", "Mar", "Apr", "Maj", "Jun", "Jul", "Aug", "Sep", "Okt", "Nov", "Dec" ]
    });

#### navigationAsDateFormat

Type: Boolean
Default: false

Whether the prevText and nextText options should be parsed as dates by the `formatDate` function, allowing them to display the target month names for example.

    $( ".selector" ).datepicker({
      navigationAsDateFormat: true
    });

#### nextText

Type: String
Default: "Next"

The text to display for the next month link. With the standard ThemeRoller styling, this value is replaced by an icon.

    $( ".selector" ).datepicker({
      nextText: "Later"
    });

#### numberOfMonths

Type: Number or Array
Default: 1

The number of months to show at once.

Multiple types supported:

- Number: The number of months to display in a single row.
- Array: An array defining the number of rows and columns to display.

例子：

    $( ".selector" ).datepicker({
      numberOfMonths: [ 2, 3 ]
    });

#### onChangeMonthYear

Type: Function( Integer year, Integer month, Object inst )
Default: null

Called when the datepicker moves to a new month and/or year. The function receives the selected year, month (1-12), and the datepicker instance as parameters. this refers to the associated input field.

#### onClose

Type: Function( String dateText, Object inst )
Default: null

Called when the datepicker is closed, whether or not a date is selected. The function receives the selected date as text ("" if none) and the datepicker instance as parameters. this refers to the associated input field.

#### onSelect

Type: Function( String dateText, Object inst )
Default: null

Called when the datepicker is selected. The function receives the selected date as text and the datepicker instance as parameters. this refers to the associated input field.

#### prevText

Type: String
Default: "Prev"

The text to display for the previous month link. With the standard ThemeRoller styling, this value is replaced by an icon.

    $( ".selector" ).datepicker({
      prevText: "Earlier"
    });

#### selectOtherMonths

Type: Boolean
Default: false

Whether days in other months shown before or after the current month are selectable. This only applies if the `showOtherMonths` option is set to true.

    $( ".selector" ).datepicker({
      selectOtherMonths: true
    });

#### shortYearCutoff

Type: Number or String
Default: "+10"

The cutoff year for determining the century for a date (used in conjunction with dateFormat 'y'). Any dates entered with a year value less than or equal to the cutoff year are considered to be in the current century, while those greater than it are deemed to be in the previous century.

Multiple types supported:

- Number: A value between 0 and 99 indicating the cutoff year.
- String: A relative number of years from the current year, e.g., "+3" or "-5".

例子：

    $( ".selector" ).datepicker({
      shortYearCutoff: 50
    });

#### showAnim

Type: String
Default: "show"

The name of the animation used to show and hide the datepicker. Use "show" (the default), "slideDown", "fadeIn", any of the jQuery UI effects. Set to an empty string to disable animation.

    $( ".selector" ).datepicker({
      showAnim: "fold"
    });

#### showButtonPanel

Type: Boolean
Default: false

Whether to display a button pane underneath the calendar. The button pane contains two buttons, a **Today** button that links to the current day, and a **Done** button that closes the datepicker. The buttons' text can be customized using the `currentText` and `closeText` options respectively.

    $( ".selector" ).datepicker({
      showButtonPanel: true
    });

#### showCurrentAtPos

Type: Number
Default: 0

When displaying multiple months via the `numberOfMonths` option, the `showCurrentAtPos` option defines which position to display the current month in.

    $( ".selector" ).datepicker({
      showCurrentAtPos: 3
    });

#### showMonthAfterYear

Type: Boolean
Default: false

Whether to show the month after the year in the header.

    $( ".selector" ).datepicker({
      showMonthAfterYear: true
    });

#### showOn

Type: String
Default: "focus"

When the datepicker should appear. The datepicker can appear when the field receives focus ("focus"), when a button is clicked ("button"), or when either event occurs ("both").

    $( ".selector" ).datepicker({
      showOn: "both"
    });

#### showOptions

Type: Object
Default: {}

If using one of the jQuery UI effects for the showAnim option, you can provide additional settings for that animation via this option.

    $( ".selector" ).datepicker({
      showOptions: { direction: "up" }
    });
