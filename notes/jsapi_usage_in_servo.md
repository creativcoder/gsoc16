Note: This is a copied from the [original](https://gist.github.com/jdm/84df9dee899a52ebdc8f) gist by jdm.

To leverage the SpiderMonkey engine as a javascript environment every application (in our case Servo) needs three things (mentioned in order of their initialization):

A JS_Runtime - The region where everything related to spidermonkey resides such as the execution contexts, variables, objects, scripts. It is also responsible for initializing the heap space used by SpiderMonkey VM.

A JS_Context - A code execution context, which provides a stack implementation that various constructs in javascript can use. Also provides support for js exeptions and exception propagation. There can be many execution contexts but only one active during js code execution. This context is not exposed to the user by any js native functions.

A JS_GlobalObject - The global object which resides at in the current execution context which all other objects can use as the global context programmatically.

The following are some of the types and methods exposed by the SpiderMonkey VM to be used in Servo.

Creating jsvals
===============
BooleanValue
DoubleValue
Int32Value
NullValue
ObjectOrNullValue
ObjectValue
PrivateValue
StringValue
UInt32Value
UndefinedValue

Converting jsvals
=================
JS_ValueToBoolean
JS_ValueToECMAInt32
JS_ValueToECMAUint32
JS_ValueToInt64
JS_ValueToNumber
JS_ValueToString
JS_ValueToUint16
JS_ValueToUint64

Types:
======
JSBool
JSClass - list of function pointers to native operations for JS object
JSContext - global shared context when interacting with VM
JSErrorFormatString
JSFunctionSpec - properties of native function to define in VM
JSHandleObject
JSNativeWrapper - pointer to native function to execute
JSObject - general JS object (ie. {}, no primitives)
JSPropertyDescriptor - descriptor for property of JS object
JSPropertyOpWrapper - pointer to native function to execute (getter/setter)
JSPropertySpec - properties of property to define on JS object
JSFreeOp
JSGCStatus
JSStrictPropertyOpWrapper - pointer to native function to execute (getter/setter)
JSString - general JS string
JSTracer - tracer state for garbage collector
JSVal - boxed value (any literal/object/primitive)
jschar - UCS2 character value
jsid - interned value

Flags:
======
JSEXN_TYPEERR
JSFUN_CONSTRUCTOR
JSGC_BEGIN
JSGC_END
JSGC_MAX_BYTES
JSREPORT_STRICT
JSREPORT_STRICT_MODE_ERROR
JSREPORT_WARNING
JSTRACE_OBJECT
JS_DEFAULT_ZEAL_FREQ

Methods:
========

* rooting/garbage collection:
JS_AddObjectRoot - root an object (ensure it will not be GCed)
JS_RemoveObjectRoot - unroot an object (relase any GC guarantees)
JS_CallTracer - trace a particular object
JS_GC - initiate a GC

* object representation interrogation:
JS_GetClass - get the JSClass from a JS object
JS_GetFunctionObject - get the JS object representation of a JSFunction
JS_GetGlobalForObject - get the global associated with a JSObject
JS_GetReservedSlot - retrieve the value stored in a reserved piece of memory for a particular JS object
JS_GetStringCharsAndLength - get the buffer and length from a string
JS_GetObjectPrototype - get the native pointer to Object.prototype
JS_ObjectIsCallable - is a JS object callable?
JS_ObjectIsDate - is a JS object a Date?
JS_ObjectIsRegExp - is a JS object a RegExp?
JS_ObjectToOuterObject

* object interrogation:
JS_AlreadyHasOwnProperty - is there a given own property on an object?
JS_ForwardGetPropertyTo - transparently redirect a GetProperty operation to another object
JS_GetProperty - get a property
JS_GetPropertyById - get a property using an interned string
JS_GetPropertyDescriptorById - get a descriptor representing a property using an interned string
JS_HasProperty - does the property exist on an object?
JS_HasPropertyById - does the property exist on an object using an interned string?
JS_GetPrototype - get the object's prototype

* value transfer
JS_ParseJSON - convert a JSON string into a JS object
JS_ReadStructuredClone - unpack a structured clone buffer
JS_WriteStructuredClone - pack a structured clone buffer

* execution:
JS_EvaluateUCScript - evaluate a UCS2 string
JS_CallFunctionValue - invoke a function

* object manipulation:
JS_DefineFunctions - define a set of native methods on an object
JS_DefineProperties - define a set of properties on an object
JS_DefineProperty - define a property on an object
JS_DefinePropertyById - define a property on an object using an interned string
JS_DeletePropertyById2 - define a property on an object using an interned string
JS_SetPrototype - mutate the prototype of a JS object

* creation:
JS_InitStandardClasses - initialize the standard global properties
JS_LinkConstructorAndPrototype - associate a constructor function with a prototype object
JS_NewFunction - create a new function
JS_NewGlobalObject - create a new global
JS_NewObject - create a new JS object
JS_NewObjectWithGivenProto - create a new JS object with a particular prototype
JS_NewObjectWithUniqueType - create a new JS object with a particular type (re: type inference)
JS_NewStringCopyN - create a new JS string, copying the provided non-UCS2 string
JS_NewUCString - create a new JS string
JS_NewUCStringCopyN - create a new JS string, copying the provided UCS2 string

* native object hook setup - used as default function pointers of JSClass:
JS_ArrayIterator
JS_ConvertStub
JS_EnumerateStub
JS_PropertyStub
JS_ResolveStub
JS_SetReservedSlot
JS_StrictPropertyStub

* vm state manipulation:
JS_ClearPendingException - suppress any exception that has not been reported yet
JS_CloneFunctionObject
JS_CompileUCFunction - compile a function body into a JSFunction
JS_ReportErrorFlagsAndNumber - immediately report a specific error (trigger any error report callback)
JS_ReportErrorNumber - immediately report a specific error
JS_ReportPendingException - immediately report any pending, unreported error
JS_RestoreFrameChain - unsuppress the last suppressed VM stack
JS_SaveFrameChain - suppress the current VM stack
JS_SetGCCallback - set a native callback to execute before GC
JS_SetGCParameter - configure various VM properties related to GC
JS_SetGCZeal - configure how often to GC
JS_SetPendingException - set a pending exception on the VM, but don't report it yet
JS_SetWrapObjectCallbacks - configure the native callbacks to execute when wrapping JS objects (for cross-compartment access)
JS_free - free memory allocated by the VM
JS_malloc - allocate memory for the VM
JS_WrapObject - wrap a JS object for cross-compartment access
JS_WrapValue - wrap a JS value for cross-compartment access

* vm state interrogation:
JS_GetRuntime - get a JSRuntime from a JSContext
JS_IsExceptionPending - determine if an exception has been set but not reported yet

