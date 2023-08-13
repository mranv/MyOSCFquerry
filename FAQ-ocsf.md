## What is OCSF?

Open Cybersecurity Schema Framework (OCSF) is an open-source effort aimed at creating a common schema for security events across the cybersecurity ecosystem. It aims to provide a standardized format and data model for logs and alerts, allowing better interoperability and consistency in the industry.

For more detailed information about OCSF, you can refer to the [Understanding OCSF whitepaper](https://github.com/ocsf/ocsf-docs/blob/main/Understanding%20OCSF.pdf).

## What Problems does OCSF solve?

OCSF addresses the challenge of lacking a common and agreed-upon format and data model for logs and alerts in the cybersecurity field. Many different models exist, but none have achieved widespread adoption. This lack of standardization makes it difficult to derive value from the data and poses challenges in areas such as detection engineering, threat hunting, analytics development, and AI. OCSF aims to solve these problems by providing a widely accepted schema that simplifies data integration and analysis.

## How does OCSF relate to other standards like STIX, Kestrel, OSSEM, OpenC2, and the Sigma taxonomy?

- **STIX**: OCSF and STIX are compatible and complementary. STIX focuses on threat intelligence, campaigns, and actors, while OCSF focuses on events representing activities on computer systems, networks, and cloud platforms with security implications. Observables from OCSF can be matched with IOCs from STIX.

- **Kestrel**: OCSF and Kestrel are compatible. Kestrel provides a threat hunting language that abstracts the "what to hunt" aspect for threat hunters. They work well together to provide a comprehensive approach.

- **OSSEM**: OSSEM is primarily focused on documenting, standardizing, and modeling security event logs. OCSF and OSSEM are separate efforts, but they both contribute to improving the standardization and consistency in the cybersecurity domain.

- **OpenC2**: OCSF and OpenC2 are compatible. OpenC2 provides a standardized language for the command and control of cyber defense technologies, enabling interoperability across various security tools and applications.

- **Sigma Taxonomy**: Details about how OCSF relates to the Sigma taxonomy are not explicitly mentioned in the provided text. However, both efforts contribute to improving the standardization and understanding of security-related data.

## How can I contribute to OCSF?

To contribute to OCSF, you can refer to the [OCSF Contribution Guide](https://github.com/ocsf/ocsf-schema/blob/main/CONTRIBUTING.md), which provides information on how to get involved, submit contributions, and participate in the OCSF community.

## What is OCSF Governance Model?

For details about the governance model of OCSF, you can refer to the [OCSF Governance](https://github.com/ocsf/governance/blob/main/Governance.md) document, which outlines the structure, decision-making processes, and roles within the OCSF project.


# Schema FAQ

This section provides answers to frequently asked questions about how to use the OCSF Schema.

## How do I create a typical OCSF event?

When creating an OCSF event, it's essential to choose the appropriate event class based on the type of event you're representing. Begin by selecting the relevant OCSF category to narrow down the options. For example, if you're dealing with an endpoint security product, you might choose an event class from the "System Activity" category, such as "File System Activity" for an antivirus (AV) product. Each event class has an `activity_id` enumeration that specifies the intended activity of the event, which can be specific to the class, such as "Logon" for the "Authentication" class in the "Identity and Access Management" category.

For endpoint security products that send alert events upon detecting malware, you should apply the "Security Control" profile, which adds important attributes to the "File System Activity" event class, such as a Malware object, a MITRE ATT&CK object, and disposition information. These profiles have their own attributes that must be populated.

If your endpoint security product also handles network security, you might choose an event class from the "Network Activity" category, such as the general "Network Activity" event class. Since the endpoint product has information about the host system, you would apply the "Host" profile in addition to the "Security Control" profile. The "Host" profile includes attributes about the device and the actor (e.g., process or user) on the host.

Every OCSF event must have all the required attributes of its event class populated, and it's recommended to populate the recommended attributes if possible. This includes attributes of embedded objects like Malware, Process, and Device.

All OCSF events have a set of required classification attributes from the "Base Event" class, including `class_uid`, `category_uid`, `activity_id`, and the derived `type_uid`. Their associated `*_name` attributes are optional.

In addition to the classification attributes, several other attributes from the "Base Event" class are required and must be populated: `time`, `metadata`, and `severity`. The `metadata` attribute is an object that includes the reporting event's `product` and associated `version`, along with the version of the OCSF schema adhered to with the event.

The originating event producer (i.e., the product) should be represented as the product in the event, and the `time` should reflect the actual occurrence time or the earliest available time known to the event producer or mapper.

While the `observables` array attribute is optional, populating it can enhance the clarity for event consumers and analysts, as it provides a common location to surface important attributes of the event in a simple tuple format: name, value, type.

## How would I populate the `observables` array?

The `observables` attribute is crucial for including important attributes of the event in a standardized way. The `Observable` object has three key attributes: `name`, `type_id`, and `value`. The first two are required, while `value` is optional, and its usage will become clearer as we delve into it.

The `name` attribute of the `Observable` object must be the fully qualified attribute name within the event. For example, it might be `fingerprint.value`, `actor.process.file`, or `actor.process.file.name`. This `observable.name` serves as the locator of that observable within the event instance. Note that the observable attribute can be a scalar, like `device.ip`, or it can be an object, like `actor.process.file`.

When the `type_id` of the observable indicates that the observable's `name` attribute is of object type (e.g., Fingerprint), the `value` attribute of the observable is not populated. Conversely, when the `type_id` indicates that the observable's `name` is a scalar (e.g., File Hash or File Name), the observable's `value` should be populated with the corresponding value from the event.

Multiple observables can be included within an event, even if they are of the same type. This is why the `observables` attribute is an array. Properly populating the `observables` array helps improve the clarity and structure of the event for consumers and analysts.

## When should I use a Security Finding event class?

A Security Finding in OCSF represents the outcome of various processes such as enrichment, correlation, aggregation, analysis, or other forms of processing applied to one or more events or alerts. It produces a derived insight based on the collected data. It's important to note that Security Findings in OCSF are not the same as alerts, although they may trigger alerts or be added to an incident downstream in the security process.

For example, consider an email security product that detects a phishing attempt or identifies a malicious email attachment. It might send an email activity event (from its perspective, an alert) containing information about the user, sender, and supplemented by the "Malware" profile with a disposition of "Blocked," along with details about the malware. This event is sent to the product's management console and then to a Security Information and Event Management (SIEM) system.

The SIEM might receive related events or alerts, such as those related to other users in a similar situation or general email activity from the same sender. The SIEM could enrich these events with data from a Threat Intelligence Platform or a threat feed related to the email sender. The resulting aggregation and enrichment process constitutes an OCSF Security Finding. The SIEM might create an incident that includes or references the finding, especially if there are remediation steps required.

It's important to recognize that, in more complex processing architectures, layered findings may exist. The original event may go to product A, which eventually triggers a finding. Meanwhile, product B might receive many other events and findings (including those from product A) and generate its own findings. This layered approach can lead to more comprehensive findings that capture the essence of the event and its significance in the broader context.

## When should I use metadata.correlation_uid?

The `metadata.correlation_uid` attribute is valuable when an event producer or mapper emits multiple events that share a grouping characteristic or have some form of similarity. By populating the `metadata.correlation_uid` attribute with a constant identifier, you make it easier for consumers and analysts to aggregate and correlate the related events.

Consider a vulnerability scanner that emits events at the beginning and end of a system scan, as well as separate events for each vulnerability discovered. If these are separate events, they should all have their `metadata.correlation_uid` set to the same value. This attribute helps establish a relationship between the events and simplifies the process of understanding the context in which they occur.

While an intermediary system can also determine the grouping characteristic and populate the `metadata.correlation_uid` attribute after collecting the events, it's important to note that OCSF events are typically immutable. If correlation information needs to be added, a copy of the original events would be created, including the additional correlation information.

## Can Finding events be correlated with each other too?

Yes, Finding events can be correlated with each other in a similar manner. Since Finding events also have a base class metadata object, you can apply the same pattern of using `metadata.correlation_uid` to tie multiple Finding events together. This allows you to establish relationships and identify correlated insights based on shared identifiers.

For

 example, a SIEM that generates findings may have sufficient knowledge and context to link multiple findings together using the `metadata.correlation_uid`. This enables the SIEM to show how different findings relate to each other, leading to a more comprehensive understanding of the overall security landscape.

## How do I use the Actor object?

The Actor object in OCSF is designed for use in event classes when there's a need to identify an entity that initiates or causes an action on another entity. It's particularly useful when representing scenarios where one process triggers another process or performs actions like file deletion. The Actor object helps avoid naming collisions with the other end of the activity, especially in cases where a process acts on another process, as attribute names could be in contention at the same level within the class.

The Actor object currently includes two key attributes: `process` and `user`. In many cases, one of these attributes is in the role of the actor in the activity. Additionally, the Actor object has optional attributes for `session`, `authorizations`, `idp` (Identity Provider), and `invoked_by`.

- The `idp` attribute is relevant in Identity and Access Management (IAM) category event classes. It is populated when the actor's identity provider is known and is logged along with Authentication and related events.

- The `authorizations` attribute represents an array of information related to the permissions and privileges the actor has at the time of the event, if that information is known.

- The `invoked_by` attribute is populated with the name of the service or application through which the actor's activity was initiated.

The Actor object helps provide structured information about the actor entity in the event, enhancing the clarity and completeness of the event representation.

## When should I use the session attribute?

The `session` attribute is typically paired with the `user` attribute and is primarily used to provide information about a specific user session through which an activity was initiated. The `Session` object contains details related to the user session, and it is especially useful when the event involves user interactions.

It's important to note that the `user` attribute, represented as a `User` object, is not always associated with a session, and it isn't always an actor. Therefore, the `Session` object is included within the Actor object to provide a dedicated structure for session-related information when necessary.

The `process` attribute, when represented as a `Process` object, also has a `user` attribute, which indicates the user's account that the process runs under or is impersonating. Consequently, the `Session` attribute is paired with the `user` attribute in many event classes, especially when capturing user-related activity.

## When should I use the unmapped attribute?

The `unmapped` attribute in OCSF serves as a catchall for event producers and mappers when there's data that doesn't fit neatly into the more specific attributes of the class. It's a way to handle product-specific data that might be extracted from logs but isn't directly mappable to the defined attributes.

The primary use case for the `unmapped` attribute is when dealing with events from multiple vendors, and each vendor may have unique fields that are not common to other vendors for the same type of data source. By using the `unmapped` attribute, you can ensure that such data is still included in the event representation, even if it doesn't align perfectly with the existing attributes.

However, it's important to note that using the `unmapped` attribute is not recommended for event producers that have control over the schema. If you're the original event producer, it's better to extend the schema to properly capture any data that can't be mapped. For product-specific data, consider using an extension, such as a vendor-developed profile or, in some cases, creating a new event class that adequately represents the event when the core event class falls short due to unmappable data or unique activities.

## unmapped is of Object type. What does that mean and is it different from JSON or a String type?

The `unmapped` attribute in OCSF is of type "Object," which is an empty complex data type from which all OCSF objects extend. This allows the `unmapped` attribute to include JSON-formatted attributes, requirements, and descriptions, making it a versatile container for data that doesn't fit into predefined attributes.

The key distinction between the `unmapped` attribute and JSON or String types is that `unmapped` is specifically designed within the OCSF schema as a structured placeholder for unmapped or vendor-specific data. It's not a free-form JSON field, and it's not simply a String type for unformatted text.

In the context of OCSF, an `unmapped` attribute can be thought of as an OCSF object that you create on the fly. It's used within the instance of an OCSF class only and is not part of the global schema. This design allows event producers and mappers to include data that doesn't conform to the schema's predefined attributes, ensuring that relevant information is captured even if it can't be directly mapped.

It's important to note that OCSF does have a JSON type, which is used for the `data` attribute, but this is typically used for other purposes and should not be used as a replacement for handling unmapped fields or vendor-specific data.

## When should I use Authorize Session from Identity and Access Management vs. Web Resource Access Activity from the Application category?

The "Authorize Session" event class in the Identity and Access Management (IAM) category and the "Web Resource Access Activity" event class from the Application category serve distinct purposes and are used in different scenarios:

- **Authorize Session (IAM Category)**: The "Authorize Session" event class should be used when you need to capture events related to the authorization or change of permissions, privileges, or roles for a security principal (user or entity). This class focuses on the activity of authorizing a session and is not directly tied to a specific resource access. It represents changes in permissions or privileges that a security principal has, typically independent of any specific resource access event. For example, when a new logon session is created, or when permissions are granted or revoked, you would use the "Authorize Session" event class to capture these authorization activities.

- **Web Resource Access Activity (Application Category)**: The "Web Resource Access Activity" event class, on the other hand, is used to capture events related to the access of web resources (such as web services or REST APIs) by a security principal. This class is focused on the actual access activity to web resources and includes details about the accessed resource, the user or entity making the request, and any relevant parameters. It represents the enforcement of authorization restrictions at access time, and it's often used to log resource access attempts and outcomes.

In summary, use the "Authorize Session" event class when you want to capture events related to changes in permissions or privileges for security principals, and use the "Web Resource Access Activity" event class when you want to capture the specific access of web resources by security principals.

## When should I use HTTP Activity vs. Web Resource Access Activity?

The "HTTP Activity" event class and the "Web Resource Access Activity" event class have distinct purposes and are used in different contexts within the OCSF schema:

- **HTTP Activity**: The "HTTP Activity" event class is intended to capture information related to the network protocol activities specifically involving HTTP. It focuses on the low-level details of HTTP interactions, such

 as HTTP requests, responses, headers, and other protocol-specific elements. This class is more about the technical aspects of HTTP communication and is suitable for capturing the protocol-level activity.

- **Web Resource Access Activity**: The "Web Resource Access Activity" event class, on the other hand, is used to capture events related to the access of web resources (such as web services, REST APIs, etc.) by security principals (users or entities). This class is focused on the access activity itself, including details about the accessed resource, the user/entity making the request, any parameters, the outcome of the access attempt, and potentially additional context. It represents the enforcement of authorization restrictions at access time.

While web resources are often accessed via HTTP, the "Web Resource Access Activity" class provides a higher-level view focused on the access of resources, whereas the "HTTP Activity" class is more suited for capturing protocol-level details. The choice between these two classes depends on the level of detail you want to capture and the specific use case you're addressing.

## Can you explain Profiles to me?

Profiles in the OCSF schema are a powerful mechanism for extending event classes and objects with a standardized set of attributes. They allow you to uniformly add a predefined set of attributes to one or more event classes or objects, enhancing the flexibility and richness of the event representation. When you apply a profile to a class or object definition, you gain the ability to include the additional attributes defined in that profile within the event.

Here's how profiles work:

1. **Attributes and Event Structure**: Event classes provide the basic structure and type of an event, while objects provide the structure of complex types. A profile is a way to specify additional attributes that may be included with an event instance.

2. **Permission to Include Attributes**: When you add a profile to the definition of an event class or object, you essentially gain permission to dynamically include the attributes defined in that profile when constructing an event instance.

3. **Usage**: To use a profile, you simply add the profile's name to the `metadata.profiles` array when constructing an event. Once the profile is applied, the event becomes a "kind of" that profile in addition to being a "kind of" the event class itself.

4. **Query and Analysis**: This allows you to query the event based on the profile or the event class, depending on the context. For example, if you apply the "Host" profile to the "HTTP Activity" class to add the `actor.process` making a request, you can query the event as a "Host" or as an "HTTP Activity," and you can include other events from the "System Activity" category that also share the same actor.

5. **Partial Inclusion**: It's important to note that you don't need to include all attributes from the profile in every event. You have the flexibility to selectively include the attributes based on the specific event instance.

6. **Extension and Overriding**: You can extend a profile within the definition of a class or object. Additionally, the profile's attributes, requirements, and descriptions can be overridden within the definition of the class or object, although this is not recommended. Attribute data types and constraints, however, cannot be overridden.

Profiles are a powerful tool for handling event diversity and evolving schema requirements. They enable you to include relevant attributes without cluttering the core schema and allow for consistent representation of events across different use cases.

## Is there a similarity between OCSF and LDAP (and X.500)?

Yes, there are similarities between the OCSF schema and the LDAP (Lightweight Directory Access Protocol) schema model, as both involve representing structured information using attributes and object classes. However, OCSF is designed with a focus on representing cybersecurity events and activities, while LDAP is primarily used for directory services and information organization.

Here are some points of similarity between OCSF and LDAP (and X.500):

1. **Attributes and Object Classes**: Both OCSF and LDAP use attributes to represent specific pieces of information and object classes to define the structure of entries or events.

2. **Schema Extensibility**: Both models allow for schema extensibility. In OCSF, you can use profiles to add attributes to event classes, and in LDAP, you can define custom attributes and object classes to extend the schema.

3. **Hierarchical Structure**: X.500 and LDAP have a hierarchical structure for organizing entries in a directory, similar to the way OCSF organizes events into categories and classes.

4. **Structured Data**: Both models aim to provide a structured way to represent data, ensuring that specific information is captured in a standardized format.

Despite these similarities, it's important to note that OCSF is specialized for representing cybersecurity events, including the categorization of events, the inclusion of security-related attributes, and the support for profiles to handle event diversity. On the other hand, LDAP is used for directory services and often involves representing information about users, groups, organizational units, and other directory-related data.

In summary, while there are conceptual similarities between OCSF and LDAP/X.500 in terms of structured data representation, their primary purposes and the domains they serve are distinct. OCSF focuses on cybersecurity event representation, while LDAP is used for directory services.
