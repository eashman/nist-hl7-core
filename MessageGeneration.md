

# Introduction #

The overall generation process consists in generating an XML message template. The template is then populated by the message population module. If the output is ER7, the xml2bar converter is used.

# How to use the message generation #

Calling the message generation is pretty simple. You need a profile and message generation context. The message generation context is an XML file to configure the generation process. We’ll go into details on how to write it.

```
// Load a Profile
Profile profile = new Profile(
        MessageGenerationClient.class.getResourceAsStream("/NIST_RSP_K23.xml"));

// Load a message generation context
HL7V2MessageGenerationContextDefinitionDocument context = HL7V2MessageGenerationContextDefinitionDocument.Factory.parse(MessageGenerationClient.class.getResourceAsStream("/generationCtxt.xml"));

// Generate the messages
MessageGeneration mg = new MessageGeneration();
MessageGenerationResult mgResult = mg.generate(profile, context);
if (mgResult.hasError()) {
    for (String error : mgResult.getErrors()) {
        System.out.println(error);
    }
} else {
    for (HL7V2Message message : mgResult.getMessages()) {
        System.out.println(message.getMessageAsString());
    }
}
```

# The Message Generation Context #

A message generation context is an XML file used to configure the message generation. The configuration allows you to:

  * Define the encoding of the generated message (ER7 or XML).
  * Provide the list of data files that will be used to populate the message.
  * Set specific value at specific location in the message.

The schema for the message generation context can be found in the core-xml-beans module. The name is !HL7V2MessageGenerationContext.xsd

## Define the Encoding ##

To set the encoding of the generated message, you have to set the value of the element Encoding. It can be either XML or ER7.

```
<context:HL7V2MessageGenerationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/generation/message"
        xmlns:generation="http://www.nist.gov/healthcare/generation"
        xmlns:message="http://www.nist.gov/healthcare/message">
        <context:Encoding>ER7</context:Encoding>
        ...
</context:HL7V2MessageGenerationContextDefinition>
```

## The Message Population Module ##

The message will be populated with random values. Those values come from a set of data files. There are 3 types of data files:

  * PRIMITIVE: It contains values that are associated to primitive elements.
  * TABLE: It contains values that are associated to primitive elements but the value must come from a table.
  * DEFAULT: It contains values that will be used if no value is available for the element. The value are defined for each primitive datatype.

The list of data files is specified in the context. You can either point to a file that is on the file system or in a jar file. The following example shows that the primitive values come from the file /data/PrimitivesV2.5.1.xml. The file is in a jar file (core-base), this is why we use the prefix classpath. This is the same thing for the default value. However for the table values, the file is on the file system so we use the prefix file.

```
<context:HL7V2MessageGenerationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/generation/message"
        xmlns:generation="http://www.nist.gov/healthcare/generation"
        xmlns:message="http://www.nist.gov/healthcare/message">
        ...
        <context:MessagePopulationModule>
                <generation:Resource Type="PRIMITIVE" Source="classpath:/data/PrimitivesV2.5.1.xml"/>
                <generation:Resource Type="TABLE" Source="file:D:/tmp/data/HL7tableV2.5.1.xml"/>
                <generation:Resource Type="DEFAULT" Source="classpath:/data/DefaultValue.xml"/>
        </context:MessagePopulationModule>
        ...
</context:HL7V2MessageGenerationContextDefinition>
```

The schemas for the data files can be found in the core-xml-beans module. The names are PrimitiveValueLibrary.xsd, TableLibrary.xsd and DefaultValueLibrary.xsd


## Set Specific Value ##

It is possible to set a specific value at a specific location in the message. In the following example, we set the value 'A specific value for OBR.1' for OBR.1 of the third ORDER\_OBSERVATION segment group.

**Even if you target ER7 message, you have to specify the segment group information because the message generation works with XML data internally.**

```
<context:HL7V2MessageGenerationContextDefinition
        xmlns:context="http://www.nist.gov/healthcare/generation/message"
        xmlns:generation="http://www.nist.gov/healthcare/generation"
        xmlns:message="http://www.nist.gov/healthcare/message">
        ...
        <context:MessageInstanceSpecificValues>
                <generation:DataValueLocationItem>
                        <generation:Location>
                                <message:SegmentGroup Name="PATIENT_RESULT">
                                        <message:SegmentGroup Name="ORDER_OBSERVATION" InstanceNumber="3">
                                                <message:Segment Name="OBR">
                                                        <message:Field Position="1"/>
                                                </message:Segment>
                                        </message:SegmentGroup>
                                </message:SegmentGroup>
                        </generation:Location>
                        <generation:Value>A specific value for OBR.1</generation:Value>
                </generation:DataValueLocationItem>
        </context:MessageInstanceSpecificValues>
</context:HL7V2MessageGenerationContextDefinition>
```