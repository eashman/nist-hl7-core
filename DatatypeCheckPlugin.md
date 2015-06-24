

# Datatype Check Plugin #

This plugin add support for specific checks for the following datatypes: HD, CE, CWE and CNE. This plugin works only for V2.

```
<context:PluginCheck Name="gov.nist.healthcare.core.validation.message.plugin.datatype.DatatypeCheckPlugin">
	<validation:Params>JSON String</validation:Params>
</context:PluginCheck>
```

## Location ##

You need to provide the location. The location is where the value(s) is located in the message. A location can return several values.

In V2 the location is expressed like `PID[2].4[3].4.1`:

  * `PID[2]`: It refers to the second instance of the PID segment.
  * `4[3]`: It refers to the third instance of the fourth field.
  * `4`: It refers to the fourth component
  * `1`: It refers to the first subcomponent

Note: There is no instance number for the component and subcomponent because they are not repeatable. Also the instance numbers can be set to `*` to express every instance. In that case the location might returns several values.

## Datatype Check ##

Check that the values at `ZHD[3].1[*]` as HD elements are valued correctly.

```
{
    "datatype": "HD",
    "location": "ZHD[3].1[*]"
}
```