# ERRORS.md
<hr>
This file defines the error codes used in the project. Each error code is associated with a specific error message and a description of the error.


### `0xFADE####` - Unimplemented Feature
Where `B0` == `0xFADE####` ->

| Error Code | Feature Name |
|------------|---------------|
| 0xFADE0000 | `expose fun malloc(int size) -> void*` |
| 0xFADE0001 | `expose fun free(void *ptr) -> void` |
| 0xFADE0002 | `expose fun realloc(void *ptr, int size) -> void*` |