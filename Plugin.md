

# Introduction #

The current message validation handles basic checks but is not suited well for domain specific checks. So we introduced a plugin framework to allow the user to write specific checks in Java.

# Implementation #

## Java Class ##

The first part of writing a specific check is to write a Java class that extends from `gov.nist.healthcare.core.validation.message.plugin.ValidationPlugin` and to implement the `validate` method.

```
    public abstract List<MessageFailure> validate(HL7Message message,
            String params, AssertionResultConstants.Enum assertionResult,
            String userComment) throws Exception;
```

The method returns a list of `MessageFailure` to indicate the validation errors. The parameters are:

  * message: The message to validate, it can be a V2 or a V3 message.
  * params: The parameter for the plugin, the format used is JSON.
  * assertionResult: It is a constant to indicate how to override the interpretation of the error. For more details see [Assertion Result](http://code.google.com/p/nist-hl7-core/wiki/MessageValidation#Assertion_Result)
  * userComment: A user comment to supplement the description if the description is too generic.

## Message Validation Context ##

The second part is to call the plugin from the message validation context.

```
<?xml version="1.0" encoding="UTF-8"?>
<context:HL7V2MessageValidationContextDefinition xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context" xmlns:validation="http://www.nist.gov/healthcare/validation">
  ...
  <context:PluginCheck Name="gov.nist.healthcare.core.validation.message.plugin.value.ValueCheckPlugin">
    <validation:Params>{"values":[{"options":{"required":true},"type":"PLAIN","text":"QBP"}],"matchAll":false,"minMatch":0,"maxMatch":null,"location":"MSH[1].9[1].1"}</validation:Params>
  </con:PluginCheck>
</context:HL7V2MessageValidationContextDefinition>
```

In this example we want to use the `ValueCheckPlugin` that checks that the value `QBP` is required at location `MSH[1].9[1].1`. You can notice that the parameters are expressed with JSON.

# An example: Pix Query plugin #

The Pix Query plugin checks that a specific patient is returned in the response of a Pix query.

## The Java Class ##

```
public class PixQueryV2CheckPlugin extends ValidationPlugin {

    @Override
    public List<MessageFailure> validate(HL7Message message, String params,
            AssertionResultConstants.Enum assertionResult, String userComment)
            throws Exception {
        List<MessageFailure> messageFailures = null;
        if (message instanceof HL7V2Message) {
            messageFailures = validateMessage((HL7V2Message) message, params,
                    assertionResult, userComment);
        }
        return messageFailures;
    }

    private List<MessageFailure> validateMessage(HL7V2Message message,
            String params, AssertionResultConstants.Enum assertionResult,
            String userComment) throws JsonParseException,
            JsonMappingException, IOException {
        PixQueryV2CheckParam vcParams = mapper.readValue(params,
                PixQueryV2CheckParam.class);
        return validate(message, vcParams.getPatients(), assertionResult,
                userComment);
    }

    private List<MessageFailure> validate(HL7V2Message message,
            List<PixPatientV2Param> patients,
            AssertionResultConstants.Enum assertionResult, String userComment) {
      ...
    }
```

The first part is to call the validate method depending on the type of the message V2 or V3. In that example we just process V2 message. The second part `validateMessage` method converts the JSON parameters into a Java object (`PixQueryV2CheckParam` in that case). The last part is to write the core of the check. The first two parts are pretty common to all plugins.

## Message Validation Context ##

The last step is to call the plugin from the message validation context.

```
<context:HL7V2MessageValidationContextDefinition xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context" xmlns:validation="http://www.nist.gov/healthcare/validation">
  ...
  <context:PluginCheck Name="gov.nist.healthcare.connectathon.pixpdq.service.validator.PixQueryCheckPlugin">
    <validation:Params>{"patients":[{"patientId":"101","assigningAuthorities":[{"namespaceId":"BLUE","universalId":"1.3.6.1.4.1.21367.3000.1.1","universalIdType":"ISO"},{"namespaceId":"GREEN","universalId":"1.3.6.1.4.1.21367.3000.1.2","universalIdType":"ISO"},{"namespaceId":"WHITE","universalId":"1.3.6.1.4.1.21367.3000.1.3","universalIdType":"ISO"}]}]}</validation:Params>
  </context:PluginCheck>
</context:HL7V2MessageValidationContextDefinition>
```

The JSON string inside the `Params` tags is the one passed to the `validate` method as the `params` variable. It will be converted to an instance of `PixQueryV2CheckParam` class.

Here is the same call but with `assertionResult` and `userComment`:


```
<context:HL7V2MessageValidationContextDefinition xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context" xmlns:validation="http://www.nist.gov/healthcare/validation">
  ...
  <context:PluginCheck Name="gov.nist.healthcare.connectathon.pixpdq.service.validator.PixQueryCheckPlugin" AssertionResult="WARNING">
    <validation:Params>{"patients":[{"patientId":"101","assigningAuthorities":[{"namespaceId":"BLUE","universalId":"1.3.6.1.4.1.21367.3000.1.1","universalIdType":"ISO"},{"namespaceId":"GREEN","universalId":"1.3.6.1.4.1.21367.3000.1.2","universalIdType":"ISO"},{"namespaceId":"WHITE","universalId":"1.3.6.1.4.1.21367.3000.1.3","universalIdType":"ISO"}]}]}</validation:Params>
    <validation:userComment>Check for patient with id 101</validation:userComment>  
  </context:PluginCheck>
</context:HL7V2MessageValidationContextDefinition>
```