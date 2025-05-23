---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/executable-jna-tmpdir.html
applies_to:
  deployment:
    self:
products:
  - id: elasticsearch
---

# Ensure JNA temporary directory permits executables [executable-jna-tmpdir]

::::{note}
This is only relevant for Linux.
::::


{{es}} uses the Java Native Access (JNA) library, and another library called `libffi`, for executing some platform-dependent native code. On Linux, the native code backing these libraries is extracted at runtime into a temporary directory and then mapped into executable pages in {{es}}'s address space. This requires the underlying files not to be on a filesystem mounted with the `noexec` option.

By default, {{es}} will create its temporary directory within `/tmp`. However, some hardened Linux installations mount `/tmp` with the `noexec` option by default. This prevents JNA and `libffi` from working correctly. For instance, at startup JNA may fail to load with an `java.lang.UnsatisfiedLinkerError` exception or with a message that says something similar to `failed to map segment from shared object`, or `libffi` may report a message such as `failed to allocate closure`. Note that the exception messages can differ between JVM versions. Additionally, the components of {{es}} that rely on execution of native code via JNA may fail with messages indicating that it is `because JNA is not available`.

To resolve these problems, either remove the `noexec` option from your `/tmp` filesystem, or configure {{es}} to use a different location for its temporary directory by setting the [`$ES_TMPDIR`](/deploy-manage/deploy/self-managed/important-settings-configuration.md#es-tmpdir) environment variable. For instance:

* If you are running {{es}} directly from a shell, set `$ES_TMPDIR` as follows:

    ```sh
    export ES_TMPDIR=/usr/share/elasticsearch/tmp
    ```

* For installs done through RPM or DEB packages, the environment variable needs to be set through the [system configuration file](setting-system-settings.md#sysconfig).
* If you are using `systemd` to run {{es}} as a service, add the following line to the `[Service]` section in a [service override file](setting-system-settings.md#systemd):

    ```text
    Environment=ES_TMPDIR=/usr/share/elasticsearch/tmp
    ```


If you need finer control over the location of these temporary files, you can also configure the path that JNA uses with the [JVM flag](elasticsearch://reference/elasticsearch/jvm-settings.md#set-jvm-options) `-Djna.tmpdir=<path>` and you can configure the path that `libffi` uses for its temporary files by setting the `LIBFFI_TMPDIR` environment variable. Future versions of {{es}} may need additional configuration, so you should prefer to set `ES_TMPDIR` wherever possible.

::::{note}
{{es}} does not remove its temporary directory. You should remove leftover temporary directories while {{es}} is not running. It is best to do this automatically, for instance on each reboot. If you are running on Linux, you can achieve this by using the [tmpfs](https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html) file system.
::::


