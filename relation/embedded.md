# Embedded Entities
You can embedd one or multiple entities inside another object using parent object table as data source. It can be achieved using
`embedd` relation type and might be helpful to perform de-composition of your entity. Such relation also allows lazy and eager (default)
loading of embedded entities, or the ability to retrieve entities separally (without loading parent model).
