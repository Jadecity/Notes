**Build**
1. cd project folder
2. make all: 生成普通编译版的elf;
3. make release_av  生成发布版的最终对象。我们只用到其中的elf文件，不用bin文件。

**Flash to drone**
1. cd update_movidius
2. sudo ./mov\_test update\_app path\_to\_elf

**Test**
1. 找一台有外接线的DRONE;
2. cd project and then make all run. 即可在控制台看到输出。
