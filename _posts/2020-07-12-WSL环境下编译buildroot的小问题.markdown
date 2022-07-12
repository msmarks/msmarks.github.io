---
layout: post
title:  "在WSL2环境下编译buildroot遇到的小问题"
date:   2022-07-12 8:26:00 +0800
slug: wsl2_buildrooot
#categories: buildroot
---

# libffi 出现 `config.log: No such file or directory`

认为libffi编译脚本在NTFS格式分区的时候，文件权限设置错误导致，解决方案是：https://github.com/libffi/libffi/issues/552
，修改`ax_enable_builddir.m4`文件：
```
-test -f $srcdir/config.log   && mv $srcdir/config.log   .
+test -f $srcdir/config.log   && cp $srcdir/config.log   .
```

# 找不到文件`/usr/bin/install: cannot stat 'buildroot/output/host/aarch64-buildroot-linux-uclibc/sysroot/usr/share/terminfo/a/ansi': No such file or directory`

https://whycan.com/t_8194.html
这个错误是编译脚本错误的把文件名设置成了ASCII码，上述帖子中给出的解决方案是：

找到 buildroot\package\ncurses\ncurses.mk 中如下内容：

```bash
define NCURSES_TARGET_CLEANUP_TERMINFO
	$(RM) -rf $(TARGET_DIR)/usr/share/terminfo $(TARGET_DIR)/usr/share/tabset
	$(foreach t,$(NCURSES_TERMINFO_FILES), \
		$(INSTALL) -D -m 0644 $(STAGING_DIR)/usr/share/terminfo/$(t) \
			$(TARGET_DIR)/usr/share/terminfo/$(t)
	)
endef
```

改成

```bash
define NCURSES_TARGET_CLEANUP_TERMINFO
	$(RM) -rf $(TARGET_DIR)/usr/share/terminfo $(TARGET_DIR)/usr/share/tabset
	$(foreach t,$(NCURSES_TERMINFO_FILES), \
		$(INSTALL) -D -m 0644 $(STAGING_DIR)/usr/share/terminfo/$(shell python3 -c \
			"import sys; t = sys.argv[1]; t = f'{ord(t[0]):x}{t[1:]}'; print(t)" ${t}) \
			$(TARGET_DIR)/usr/share/terminfo/$(t)
	)
endef
```