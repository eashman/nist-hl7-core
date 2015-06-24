The NIST HL7 core library is a JAVA library to manipulate, validate and generate HL7 messages. This is a multi-module maven project including the following modules:
  * **core-xml-beans** contains all the schemas used by other modules. The schemas are transformed into Java using XMLBeans.
  * **core-base** contains all the components needed to manipulate basic HL7 objects like a profile, generic V2 XML/ER7 and V3 messages.
  * **core-xml2bar** transforms a V2 XML message into a V2 ER7 message. It is mainly used by the generation module.
  * **core-message-generation** generates HL7 V2 messages based on a profile.
  * **core-message-validation** validates HL7 V2 and V3 messages. It validates the structure and the content of the message.

Documentation is available for [generating a message](MessageGeneration.md) and [validating a message](MessageValidation.md).

**05/26/2011**

Add source into mercurial repository.

Add documentation for the new [validation plugin](Plugin.md) framework.

**06/09/2011**

Add documentation for the [data value check plugin](DataValueCheckPlugin.md) and [datatype check plugin](DatatypeCheckPlugin.md).