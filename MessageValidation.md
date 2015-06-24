

# Introduction #

The library validates the structure and the content of the messages. The structure is only for V2 message. The following checks are done for the structure:

  * Version: The version in the message (MSH.12.1) must match the version in the profile.
  * Message Type: The message type (3 components of element MSH.9) in the message must match the message code, the trigger event and the message structure id in the profile.
  * Message Structure: The order of the message element must match the definition found in the profle.
  * Usage: Related to the message structure, the validation checks for the presence or not of the message elements.
  * Cardinality: Related to the message structure, the validation checks for the cardinality of the message elements.
  * Length: The value length in the message must be less than the maximum length specified in the profile.
  * Table: The value must come from a table set.
  * Constant: The value must match the constant value defined in the profile.
  * Datatype: Thae value must match a regular expression depending on the datatype defined in the profile.

and the content:

  * Plain Text: The value at a specific location must match the value defined in the context.
  * Regular Expression: The value at a specific location must match with the regular expression in the context.
  * Conditional: The context supports conditional checks (if then else).
  * Specific Datatype: The validation makes specific checks for some of the datatypes. Only for V2 messages.
  * Multi Message Correlation: The value at a specific location must match the value coming from another message.

# How to use the message validation #

This sample client validates a HL7V2 message encoded in ER7. To validate V3 messages, you have to use the same class but they are suffixed by V3 instead of V2 (e.g. MessageValidationV3 instead of MessageValidationV2). Also the table part is not applicable to V3.

```
// Create a Profile
Profile profile = new Profile(
        MessageValidationClient.class.getResourceAsStream("/structure/v2/RSP_K22.xml"));

// Create a default MessageValidationContext
MessageValidationContextV2 context = new MessageValidationContextV2();

// Set the length error interpreted as ERROR, they are ignored by
// default
context.setFailureResult(AssertionTypeV2Constants.LENGTH,
        AssertionResultConstants.ERROR);

// Load local table file
TableLibraryDocument tableLibraryDocument = TableLibraryDocument.Factory.parse(MessageValidationClient.class.getResourceAsStream("/structure/v2/LocalTable.xml"));

// Include all the tables used for the validation
List<TableLibraryDocument> tableLibraryDocuments = new ArrayList<TableLibraryDocument>();
tableLibraryDocuments.add(tableLibraryDocument);
tableLibraryDocuments.add(TableLibraryDocument.Factory.parse(MessageValidationClient.class.getResourceAsStream(Constants.getHl7Tables(profile.getHl7VersionAsString()))));

// Load an ER7 message
Er7Message er7 = new Er7Message(new File(getClass().getResource(
        "/structure/v2/er7/TestValid.er7").getFile()));

// Create the validation
MessageValidationV2 validator = new MessageValidationV2();

// Validate
MessageValidationResultV2 mvr = validator.validate(er7, profile,
        context, tableLibraryDocuments);

// Save the report
HL7V2MessageValidationReportDocument report = mvr.getReport();
report.save(new File("report.xml"));
```

# The Message Validation Context for V2 messages #

## Table Value Checking ##

The validation can be configured for table value checking. The profile associates some elements with specific table. It is possible to decide which tables the user wants to check for. It can be done for HL7 tables and User tables. You can ask for a list of tables or all or none of the tables.

### List of Tables ###

In this example we want to check for all HL7 tables but only user table 0300.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        <context:ValidationConfiguration>
                <context:HL7Tables>
                        <context:All/>
                </context:HL7Tables>
                <context:UserTables>
                        <context:UserTable>0300</context:UserTable>
                </context:UserTables>
        </context:ValidationConfiguration>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

### All Tables ###

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        <context:ValidationConfiguration>
                <context:HL7Tables>
                        <context:All/>
                </context:HL7Tables>
                <context:UserTables>
                        <context:All/>
                </context:UserTables>
        </context:ValidationConfiguration>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

### None of the Tables ###

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        <context:ValidationConfiguration>
                <context:HL7Tables>
                        <context:None/>
                </context:HL7Tables>
                <context:UserTables>
                        <context:None/>
                </context:UserTables>
        </context:ValidationConfiguration>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

## Error Interpretation ##

The second part of configuration defines the interpretation of the different errors. Each type of error can be set to four different levels. The type of errors are:

  * MESSAGE\_STRUCTURE: The message structure is broken. It can be a missing segment, an extra element or a misspelled element (only XML message).
  * USAGE: A required element is missing. This type of error is more precise than the MESSAGE\_STRUCTURE but the meaning is quite the same; the message structure is broken.
  * X-USAGE: An unsupported element is present.
  * XTRA: An extra element (not defined in the profile) is present.
  * CARDINALITY: An element is present a number of times less than the minimum cardinality or greater than the maximum cardinality specified in the profile.
  * LENGTH : The length of a value is greater than the maximum length specified in the profile.
  * DATATYPE: The value is not following the regular expression defined by the standard for certain datatype.
  * DATA: The value does not match a constant specified in the profile. The value does not match a value defined in a table.

The different levels are:

  * ERROR: This is an error, a message is considered valid if the number of error is 0.
  * WARNING: The error is flagged as a warning.
  * IGNORE: The error is ignored. For instance you can ignore the LENGTH error, even if there are some errors of this type, the message will be considered as valid.
  * ALERT: This level is to distinguish a real error, and an error due to a user input error like not provided a local table, or specifying a location in the message that does not exist.
  * AFFIRMATIVE: This level is to report any positive checks done during the validation.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        <context:FailureInterpretation>
                <context:MessageFailure Type="MESSAGE_STRUCTURE" Result="ERROR"/>
                <context:MessageFailure Type="USAGE" Result="ERROR"/>
                <context:MessageFailure Type="X-USAGE" Result="ERROR"/>
                <context:MessageFailure Type="XTRA" Result="ERROR"/>
                <context:MessageFailure Type="CARDINALITY" Result="ERROR"/>
                <context:MessageFailure Type="LENGTH" Result="IGNORE"/>
                <context:MessageFailure Type="DATATYPE" Result="ERROR"/>
                <context:MessageFailure Type="DATA" Result="ERROR"/>
        </context:FailureInterpretation>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

## Validation Rules ##

The last part of configuration allows the user to make sure that a set of values has been set at a specific message location. Before going into the details of the configuration, we'll explain how to express a message location. Let's take the message location `PID[2].4[3].4.1`. This location refers to the first subcomponent, of the fourth component, of the third instance of the fourth field, of the second instance of the PID segment.

  * `PID[2]`: It refers to the second instance of the PID segment.
  * `4[3]`: It refers to the third instance of the fourth field.
  * 4: It refers to the fourth component
  * 1: It refers to the first subcomponent
Note: There is no instance number for the component and subcomponent because they are not repeatable.

### PlainText ###

In the following rules, we want to check that `EVN[1].5[1].16.3` is equal to A or B. The segment group is necessary for XML encoded messages. The instance number for Segment and Field have been ommited, in that case the instance number is considered to be 1. The PlainText element have two attributes. ignoreCase which ignore the case when doing the comparison. interpretAsNumber which converts the value in the context and in the message to a decimal, and then do the comparison.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>B</validation:PlainText>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### Regular Expression ###

In the following rules, we want to check that `EVN[1].5[1].16.3` is equal to A or B but with a regular expression `[AB]`.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:RegEx>[AB]</validation:RegEx>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### Presence ###

In the following rules, we want to check that `EVN[1].5[1].16.3` is not present in the message.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:Empty/>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

In the following rules, we want to check that `EVN[1].5[1].16.3` is present in the message.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:Present/>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### Assertion Result ###

It is possible to override the category in which the assertion will be categorized. In the following context, in a message where `EVN[1].5[1].16.3` is not present, the generation would have categorized the assertion as an ERROR, but in this case it will be a WARNING.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem AssertionResult="WARNING">
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:Present/>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### Single Message Correlation ###

In the following rules, we want to check that `PID[1].3[1].4.1` = `MRG[1].1[1].4.1`. The second part of the assertion must be present in the message. If it's not present, the validation will return a MESSAGE\_VALIDATION\_CONTEXT alert instead of a DATA error.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:DataValueLocationItem>
                <context:Location>
                        <message:Segment Name="PID">
                                <message:Field Position="3">
                                        <message:Component Position="4">
                                                <message:SubComponent Position="1"/>
                                        </message:Component>
                                </message:Field>
                        </message:Segment>
                </context:Location>
                <context:Value>
                        <validation:Location>
                                <message:Segment Name="MRG">
                                        <message:Field Position="1">
                                                <message:Component Position="4">
                                                        <message:SubComponent Position="1"/>
                                                </message:Component>
                                        </message:Field>
                                </message:Segment>
                        </validation:Location>
                </context:Value>
        </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### AnyInstanceNumber ###

In the following rules, we introduce the wildcard character when we don't know the instance number. We want to check that `EVN[*].5[*].16.3` is equal to A or B. The validation will look into every EVN segment and EVN.5 field and will stop when finding one that have component #16 and subcomponent #3 that is equal to A or B. If the InstanceNumber attribute is set, the AnyInstanceNumber attribute is ignored.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN" AnyInstanceNumber="true">
                                                <message:Field Position="5" AnyInstanceNumber="true">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:Empty/>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### CheckAll ###

In the following rules, we want to check that all the values at the location `EVN[*].5[1].16.3` is equal to A9 or match the regular expression B\d or that it is present. You have to set the attribute CheckAll to true in the Location element to use this feature.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:Location>
                                <SegGroup Name="GROUP1">
                                        <message:Segment Name="EVN" AnyInstanceNumber="true">
                                                <message:Field Position="5">
                                                        <message:Component Position="16">
                                                                <message:SubComponent Position="3"/>
                                                        </message:Component>
                                                </message:Field>
                                        </message:Segment>
                                </SegGroup>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:RegEx>B\d</validation:RegEx>
                        </context:Value>
                        <context:Value>
                                <validation:RegEx>.+</validation:RegEx>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V2MessageValidationContextDefinition>
```

### Conditional Check ###

In the following rules, we want to check that `OBX[1].5[1]` is A or B if the value of `OBX[1].2[1]` is ST or ID. If the value of `OBX[1].2[1]` is set to a value different from ST or ID, you want to check that the value of `OBX[1].5[1]` is C or D. The mechanism is similar to a if-then-else pattern.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:IfThenElse>
                <context:If>
                        <context:Location>
                                <message:Segment Name="OBX">
                                        <message:Field Position="2"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>ST</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>ID</validation:PlainText>
                        </context:Value>
                </context:If>
                <context:Then>
                        <context:Location>
                                <message:Segment Name="OBX">
                                        <message:Field Position="5"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>B</validation:PlainText>
                        </context:Value>
                </context:Then>
                <context:Else>
                        <context:Location>
                                <message:Segment Name="OBX">
                                        <message:Field Position="5"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>C</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>D</validation:PlainText>
                        </context:Value>
                </context:Else>
        </context:IfThenElse>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

### MatchingSegmentInstanceNumber and MatchingFieldInstanceNumber ###

The attributes MatchingSegmentInstanceNumber and MatchingFieldInstanceNumber allow the user to do conditional check even if he doesn't know the instance number. If you want to do the same checks as in the previous section but this time you don't know the instance number, you have to use the attribute MatchingSegmentInstanceNumber and MatchingFieldInstanceNumber. Basically the validation will make sure that the check is done within the same instance of segment and field.

Let's say you have a message with `OBX[1].2[1]` = ST, `OXB[1].5[1]` = D, `OXB[2].5[1]` = B. The validation will do the if-then-else pattern for each segment, so if `OBX[1].2[1]` = ST then `OXB[1].5[1]` = A or B? and if `OBX[2].2[1]` = ST then `OXB[2].5[1]` = A or B?. The first check is not valid but the second is. The validation will not flag the first error since the second check is valid.

Anytime you have an AnyInstanceNumber in the location you might want to use these two attributes.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:IfThenElse MatchingSegmentInstanceNumber="true" MatchingFieldInstanceNumber="true">
                <context:If>
                        <context:Location>
                                <message:Segment Name="OBX" AnyInstanceNumber="true">
                                        <message:Field Position="2"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>ST</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>ID</validation:PlainText>
                        </context:Value>
                </context:If>
                <context:Then>
                        <context:Location>
                                <message:Segment Name="OBX" AnyInstanceNumber="true">
                                        <message:Field Position="5"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>B</validation:PlainText>
                        </context:Value>
                </context:Then>
                <context:Else>
                        <context:Location>
                                <message:Segment Name="OBX" AnyInstanceNumber="true">
                                        <message:Field Position="5"/>
                                </message:Segment>
                        </context:Location>
                        <context:Value>
                                <validation:PlainText>C</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>D</validation:PlainText>
                        </context:Value>
                </context:Else>
        </context:IfThenElse>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

### Specific Datatype Checks ###

The validation supports specific checks for the following datatypes: HD, CE, CWE and CNE.

In the following rule, we want to check that `MSH[1].3[1]` as a HD element is valued correctly.

```
<context:HL7V2MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v2/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:DatatypeCheck Datatype="HD">
                <context:Location>
                        <message:Segment Name="MSH">
                                <message:Field Position="3"/>
                        </message:Segment>
                </context:Location>
        </context:DatatypeCheck>
        ...
</context:HL7V2MessageValidationContextDefinition>
```

### Multi Message Correlation ###

A typical scenario for using multi message correlation checks, is when you want to check a response message where some values must match values from the request. For instance the QPD segment must be the same in the response as the one in the request.

The first step is to define the checks in a file as follows:

We want to check that the value at the location `MSH[1].3[1].1` is the same as the value at the location `MSH[1].5[1].1` in the query message. `query` is the id of the message, we'll see how to define the id of messages later.
```
<MessageValidationCorrelation
        xmlns="http://www.nist.gov/healthcare/validation/message/hl7/context">
        <Correlations>
                <Correlation>
                        <SingleValue>
                                <Between>MSH[1].3[1].1</Between>
                                <And messageKey="query">MSH[1].5[1].1</And>
                        </SingleValue>
                </Correlation>
        </Correlations>
</MessageValidationCorrelation>
```

We want to check that all the values starting from `QPD[1]` are the same as the values in the query message.
```
<MessageValidationCorrelation
        xmlns="http://www.nist.gov/healthcare/validation/message/hl7/context">
        <Correlations>
                <Correlation>
                        <Copy>
                                <Source messageKey="query">QPD[1]</Source>
                        </Copy>
                </Correlation>
        </Correlations>
</MessageValidationCorrelation>
```

After defining the correlations, you need to call a method that will process that file and add the checks to an empty or an existing message validation context. In the correlation document you defined ids for the messages. The ids are set when you add the messages to the HashMap. In the code below, we have 2 messages, one with the id `query` and the other one with the id `response`. Then the call to `addValidationChecks`, processes the correlation document and adds the checks to the validation context. The last step is to use the validation context to validate the message.

```
// Empty Context
MessageValidationContextV2 context = new MessageValidationContextV2();
context.load(new File(getClass().getResource(
        "/content/v2/EmptyContext.xml").getFile()));
// Correlation Document
MessageValidationCorrelationDocument correlationDoc = MessageValidationCorrelationDocument.Factory.parse(MessageContentValidationV2Test.class.getResourceAsStream("/content/v2/TestMultiMessageCorrelationDocument.xml"));
// List of messages
Map<String, HL7V2Message> messages = new HashMap<String, HL7V2Message>();
Er7Message query = new Er7Message(
        new File(
                MessageContentValidationV2Test.class.getResource(
                        "/content/v2/TestMultiMessageCorrelationQuery.er7").getFile()));
Er7Message response = new Er7Message(
        new File(
                MessageContentValidationV2Test.class.getResource(
        "/content/v2/TestMultiMessageCorrelationResponse.er7").getFile()));
messages.put("query", query);
messages.put("response", response);
// Profile
Profile profile = new Profile(getClass().getResourceAsStream(
"/content/v2/NIST_RSP_K23.xml"));
context.addValidationChecks(correlationDoc, messages, profile);
```

# The Message Validation Context for V3 messages #

## Error Interpretation ##

The first part of configuration defines the interpretation of the different errors. Each type of error can be set to four different levels. The only type of error is:

  * DATA: The message does not follow the rules defined in the message validation context.

The different levels are:

  * ERROR: This is an error, a message is considered valid if the number of error is 0.
  * WARNING: The error is flagged as a warning.
  * IGNORE: The error is ignored. For instance you can ignore the LENGTH error, even if there are some errors of this type, the message will be considered as valid.
  * ALERT: This level is to distinguish a real error, and an error due to a user input error like not provided a local table, or specifying a location in the message that does not exist.
  * AFFIRMATIVE: This level is to report any positive checks done during the validation.

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        <context:FailureInterpretation>
                <context:MessageFailure Type="DATA" Result="ERROR"/>
        </context:FailureInterpretation>
        ...
</context:HL7V3MessageValidationContextDefinition>
```

## Validation Rules ##

### Plain Text ###

In the following rules, we want to check that one of the values at `/*:PRPA_IN201301UV02/*:processingCode/@code` is equal to A or B.

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:XPath>/*:PRPA_IN201301UV02/*:processingCode/@code</context:XPath>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>B</validation:PlainText>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V3MessageValidationContextDefinition>
```

### Regular Expression ###

In the following rules, we want to check that one the values at `/*:PRPA_IN201301UV02/*:processingCode/@code` is equal to A or B but with a regular expression `[AB]`.

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:XPath>/*:PRPA_IN201301UV02/*:processingCode/@code</context:XPath>
                        <context:Value>
                                <validation:RegEx>[AB]</validation:RegEx>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V3MessageValidationContextDefinition>
```

### Presence ###

In the following rules, we want to check that `/*:PRPA_IN201301UV02/*:processingCode/@code` is not present in the message. You can also check that an element is present by using the regular expression `.+`

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:XPath>/*:PRPA_IN201301UV02/*:processingCode/@code</context:XPath>
                        <context:Value>
                                <validation:Empty/>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V3MessageValidationContextDefinition>
```

### CheckAll ###

In the following rules, we want to check that all the values at the location `/*:PRPA_IN201301UV02/*:processingCode/@code` is equal to A9 or match the regular expression `B\d` or that it is present. You have to set the attribute CheckAll to true in the Location element to use this feature.

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:MessageInstanceSpecificValues>
                <context:DataValueLocationItem>
                        <context:XPath>/*:PRPA_IN201301UV02/*:processingCode/@code</context:XPath>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:RegEx>B\d</validation:RegEx>
                        </context:Value>
                        <context:Value>
                                <validation:RegEx>.+</validation:RegEx>
                        </context:Value>
                </context:DataValueLocationItem>
        ...
        <context:MessageInstanceSpecificValues>
</context:HL7V3MessageValidationContextDefinition>
```

### Conditional Check ###

In the following rules, we want to check that that the value at `/*:PRPA_IN201301UV02/*:receiver/*:device/*:telecom/@value` is A or B if the value at `/*:PRPA_IN201301UV02/*:processingCode/@code` is ST or ID. If the value at `/*:PRPA_IN201301UV02/*:processingCode/@code` is set to a value different from ST or ID, you want to check that the value at `//*:asOtherIDs/*:id/@root` is C or D. The mechanism is similar to a if-then-else pattern.

```
<context:HL7V3MessageValidationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/validation/message/hl7/v3/context"
        xmlns:message="http://www.nist.gov/healthcare/message"
        xmlns:validation="http://www.nist.gov/healthcare/validation">
        ...
        <context:IfThenElse>
                <context:If>
                        <context:XPath>/*:PRPA_IN201301UV02/*:processingCode/@code</context:XPath>
                        <context:Value>
                                <validation:PlainText>ST</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>ID</validation:PlainText>
                        </context:Value>
                </context:If>
                <context:Then>
                        <context:XPath>/*:PRPA_IN201301UV02/*:receiver/*:device/*:telecom/@value</context:XPath>
                        <context:Value>
                                <validation:PlainText>A</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>B</validation:PlainText>
                        </context:Value>
                </context:Then>
                <context:Else>
                        <context:XPath>//*:asOtherIDs/*:id/@root</context:XPath>
                        <context:Value>
                                <validation:PlainText>C</validation:PlainText>
                        </context:Value>
                        <context:Value>
                                <validation:PlainText>D</validation:PlainText>
                        </context:Value>
                </context:Else>
        </context:IfThenElse>
        ...
</context:HL7V3MessageValidationContextDefinition>
```

### Multi Message Correlation ###

A typical scenario for using multi message correlation checks, is when you want to check a response message where some values must match values from the request. For instance the sender in the query message must be the same as the receiver in the response message.

The first step is to define the checks in a file as follows:

We want to check that the value at the location `//*:receiver/*:device/*:id/@root` is the same as the value at the location `//*:sender/*:device/*:id/@root` in the query message. `query` is the id of the message, we'll see how to define the id of messages.
```
<MessageValidationCorrelation
        xmlns="http://www.nist.gov/healthcare/validation/message/hl7/context">
        <Correlations>
                <Correlation>
                        <SingleValue>
                                <Between>//*:receiver/*:device/*:id/@root</Between>
                                <And messageKey="query">//*:sender/*:device/*:id/@root</And>
                        </SingleValue>
                </Correlation>
        </Correlations>
</MessageValidationCorrelation>
```

We want to check that the all the values starting from `//*:controlActProcess/*:queryByParameter` are the same as the values in the query message.
```
<MessageValidationCorrelation
        xmlns="http://www.nist.gov/healthcare/validation/message/hl7/context">
        <Correlations>
                <Correlation>
                        <Copy>
                                <Source messageKey="query">//*:controlActProcess/*:queryByParameter</Source>
                        </Copy>
                </Correlation>
        </Correlations>
</MessageValidationCorrelation>
```

After defining the correlations, you need to call a method that will process that file and add the checks to an empty or an existing message validation context. In the correlation document you defined ids for the messages. The ids are set when you add the messages to the HashMap. In the code below, we have 2 messages, one with the id `query` and the other one with the id `response`. Then the call to `addValidationChecks`, processes the correlation document and adds the checks to the validation context. The last step is to use the validation context to validate the message.

```
// Empty Context
MessageValidationContextV3 context = new MessageValidationContextV3();
context.load(new File(getClass().getResource(
        "/content/v3/EmptyContext.xml").getFile()));
// Correlation Document
MessageValidationCorrelationDocument correlationDoc = MessageValidationCorrelationDocument.Factory.parse(MessageContentValidationV3Test.class.getResourceAsStream("/content/v3/TestMultiMessageCorrelationDocument.xml"));
// List of messages
Map<String, HL7V3Message> messages = new HashMap<String, HL7V3Message>();
HL7V3Message query = new HL7V3Message(
        new File(
                MessageContentValidationV3Test.class.getResource(
                        "/content/v3/TestMultiMessageCorrelationQuery.xml").getFile()));
HL7V3Message response = new HL7V3Message(
        new File(
                MessageContentValidationV3Test.class.getResource(
        "/content/v3/TestMultiMessageCorrelationResponse.xml").getFile()));
messages.put("query", query);
messages.put("response", response);
context.addValidationChecks(correlationDoc, messages);
```