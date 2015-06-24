

# Data Value Check Plugin #

This plugin checks specific values in a message at a specific location.

```
<context:PluginCheck Name="gov.nist.healthcare.core.validation.message.plugin.value.ValueCheckPlugin">
	<validation:Params>JSON String</validation:Params>
</context:PluginCheck>
```

## Location ##

You need to provide the location. The location is where the value(s) is located in the message. It is expressed differently in V2 and V3. A location can return several values.

In V2 the location is expressed like `PID[2].4[3].4.1`:

  * `PID[2]`: It refers to the second instance of the PID segment.
  * `4[3]`: It refers to the third instance of the fourth field.
  * `4`: It refers to the fourth component
  * `1`: It refers to the first subcomponent

Note: There is no instance number for the component and subcomponent because they are not repeatable. Also the instance numbers can be set to `*` to express every instance. In that case the location might returns several values.

In V3 the location is express as an XPath expression.

## List of value checks ##

The list of value checks is the list of checks you want to apply to the values returned by the location. There are 5 checks:

### Plain Text (PLAIN) ###

You want to check against a plain text value. It has theses options:
  * ignoreCase: The case can be ignored.
  * interpretAsNumber: The value is converted into a number before being compared.
  * required: The location must return a value.

Check that the value at `PV1[1].3[1].4.1` matches the text `NorTh_HOspItal`. The case is ignored.

```
{
    "values": [
        {
            "options": {
                "ignoreCase": true
            },
            "type": "PLAIN",
            "text": "NorTh_HOspItal"
        }
    ],
    "matchAll": false,
    "minMatch": 0,
    "maxMatch": null,
    "location": "PV1[1].3[1].4.1"
}
```

### Regular Expression (REGEX) ###

You want to check against a regular expression. It has this option:
  * required: The location must return a value.

Check that the value at `OBR[2].1[1]` is a single digit.

```
{
    "values": [
        {
            "options": {},
            "type": "REGEX",
            "text": "\\d"
        }
    ],
    "matchAll": false,
    "minMatch": 0,
    "maxMatch": null,
    "location": "OBR[2].1[1]"
}
```

### Single Message Correlation (LOCATION) ###

You want to check against a value specified by another location inside the message. The same options as plain text can be set.

Check that the value at `PID[1].1[1]` is the same as the value at `OBR[1].1[1]`.

```
{
    "values": [
        {
            "options": {},
            "type": "LOCATION",
            "text": "OBR[1].1[1]"
        }
    ],
    "matchAll": false,
    "minMatch": 0,
    "maxMatch": null,
    "location": "PID[1].1[1]"
}
```

### Presence (PRESENT, EMPTY) ###

You want to check if a value is present or not in the message.

Check that the value at `MSH[1].8[1]` is empty.

```
{
    "values": [
        {
            "options": {},
            "type": "EMPTY",
            "text": null
        }
    ],
    "matchAll": false,
    "minMatch": 0,
    "maxMatch": null,
    "location": "MSH[1].8[1]"
}
```

## Combination of several checks ##

You can specify several checks and combine them. This is done by the boolean `matchAll` property.

  * true: In that case the value(s) must be valid against all checks. It’s interpreted as a AND.
  * false: In that case the value(s) must be valid against at least one of the checks. It’s interpreted as a OR.

Check that the value at `OBR[1].1[1]` is a single digt AND equals to `1`.

```
{
    "values": [
        {
            "options": {},
            "type": "REGEX",
            "text": "\\d"
        },
        {
            "options": {},
            "type": "PLAIN",
            "text": "1"
        }
    ],
    "matchAll": true,
    "minMatch": 0,
    "maxMatch": null,
    "location": "OBR[1].1[1]"
}
```

## Cardinality Checks ##

When a location returns several values, you want to be able to check if a certain number of them are valid. This is done by the `minMatch` and `maxMatch` properties.

  * minMatch: It’s an Integer that represents the minimum number of value that should be valid to make the whole check valid.
  * maxMatch: It’s a String that represents the maximum number of value that should be valid to make the whole check valid.

Check that at least one of the values at `OBX[*].3[1].1` is equals to `25186-8`.

```
{
    "values": [
        {
            "options": {},
            "type": "PLAIN",
            "text": "25186-8"
        }
    ],
    "matchAll": false,
    "minMatch": 1,
    "maxMatch": "*",
    "location": "OBX[*].3[1].1"
}
```