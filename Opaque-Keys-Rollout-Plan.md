## 1. Use OpaqueKeys library in LMS and Studio (no externally visible changes in LMS)
This first phase is currently implement on the opaque-keys branch (https://github.com/edx/edx-platform/pull/2905). In this phase, the older key format (`org/course/run` and `i4x://org/course/category/name@version`) will be serialized/deserialized at the system extremities. The external interfaces to the LMS won't change (this includes external apis such as LTI, and XQueue, the relational database, event logs, and html ids). Studio URL formats will be upgraded to the new key serialization schema.

## 2. Extract OpaqueKeys library and plugins
The `opaque_keys` library, the `CourseKey`, `UsageKey`, `DefinitionKey`, and `AssetKey` base classes, and the `locations.py` and `locators.py` implementations of those base classes will be extracted into a separately installable library. This library will be published to Github and PyPI, and included into the LMS and Studio via a pip installation dependency.

## 3. Use OpaqueKeys serialization format in LMS urls
This phase depends on 2., and may be merged only after giving the research community sufficient notice so that they can update their code to use the `OpaqueKeys` library.

In this phase, the LMS will be updated to support the new OpaqueKeys serialization format in incoming urls, and to send keys using the new serialization format for new key types (`*Locator` keys) when communicating to 3rd parties (XQueue, LTI, relational database).

## 4. Decompose keys in the event log
In this phase, the data accessible using the `OpaqueKeys` api will be explicitly represented in the event context in emitted events.

## 5. Decompose keys in XBlocks
In this phase, the data accessible using the `OpaqueKeys` api will be made available to XBlocks via an XBlock service.