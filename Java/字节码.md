# Class



```
ClassFile {
    u4 magic;
    u2 minor_version;
    u2 major_version;
    u2 constant_pool_count;
    cp_info constant_pool[constant_pool_count-1];
    u2 access_flags;
    u2 this_class;
    u2 super_class;
    u2 interfaces_count;
    u2 interfaces[interfaces_count];
    u2 fields_count;
    field_info fields[fields_count];
    u2 methods_count;
    method_info methods[methods_count];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```





```
method_info {
    u2 access_flags;
    u2 name_index;
    u2 descriptor_index;
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```









```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

00 09 00 00 00 1D 00



```javascript
CA FE BA BE 00 00 00 36 00 13 0A 00 04 00 0F 09 
00 03 00 10 07 00 11 07 00 12 01 00 01 6D 01 00 
01 49 01 00 06 3C 69 6E 69 74 3E 01 00 03 28 29 
56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65 4E 
75 6D 62 65 72 54 61 62 6C 65 01 00 03 69 6E 63 
01 00 03 28 29 49 01 00 0A 53 6F 75 72 63 65 46 
69 6C 65 01 00 0E 54 65 73 74 43 6C 61 73 73 2E 
6A 61 76 61 0C 00 07 00 08 0C 00 05 00 06 01 00 
1F 63 6F 6D 2F 65 78 61 6D 70 6C 65 2F 6A 76 6D 
2F 63 6C 61 7A 7A 2F 54 65 73 74 43 6C 61 73 73 
01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 
65 63 74 00 21 00 03 00 04 00 00 00 01 00 02 00 
05 00 06 00 00 00 02 00 01 00 07 00 08 00 01 00 
09 00 00 00 1D 00 01 00 01 00 00 00 05 2A B7 00 
01 B1 00 00 00 01 00 0A 00 00 00 06 00 01 00 00 
00 03 00 01 00 0B 00 0C 00 01 00 09 00 00 00 1F 
00 02 00 01 00 00 00 07 2A B4 00 02 04 60 AC 00 
00 00 01 00 0A 00 00 00 06 00 01 00 00 00 08 00 
01 00 0D 00 00 00 02 00 0E  
```

