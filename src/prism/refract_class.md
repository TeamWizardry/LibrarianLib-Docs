# `@RefractClass`

Okay, this is a really deep subject that I'm not going to fully document at the moment, however I'll
give some pointers.

- `@RefractClass`  
  Annotate your class with this to tell Prism that you want it to go through automatic 
  serialization.
- `@Refract`  
  Annotate your field with this to mark it for automatic serialization.
- `@RefractGetter`/`@RefractSetter`  
  Annotate your getters and setters with this to mark them for automatic serialization. A getter 
  with no setter is treated similarly to a final field.
- `@RefractConstructor`  
  If you want Prism to be able to create instances from scratch (necessary if there are final 
  `@Refract` fields), create a constructor with parameters that have identical names and types to
  all your `@Refract` fields. (You *need* to compile with parameter names here. Add `-parameters` to
  the Java compiler options or `-java-parameters` to the Kotlin compiler options)
- `@RefractUpdateTest`  
  A somewhat technical annotation which influences the creation of new instances.