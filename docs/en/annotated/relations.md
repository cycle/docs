# Relations

The Cycle Annotated package provides multiple annotations designed to describe entity relations. Each relation must be
associated with specific entity properties in order to work. In addition, most of the relation options (such as the name
of inner, outer keys) will be generated automatically.

> You can read more about relation configuration and usage in later sections.

## Common Statement

Each relation must have a proper `target` option. The target must point to either the related entity `role`, or to the
class name. You are able to specify class names in a fully qualified (Namespace\Class) or using current entity namespace
as the base path. You can use both `/` and `\` namespace separators.

## Relations

- [Embedded](/docs/en/relation/embedded.md)
- [BelongsTo](/docs/en/relation/belongs-to.md)
- [RefersTo](/docs/en/relation/refers-to.md)
- [HasOne](/docs/en/relation/has-one.md)
- [HasMany](/docs/en/relation/has-many.md)
- [ManyToMany](/docs/en/relation/many-to-many.md)
- [Morphed](/docs/en/relation/morphed.md)
