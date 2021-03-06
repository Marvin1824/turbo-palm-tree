- [最低要求](#minimum-requirements)
- [存储器](#storage)
- [CPU 和内存](#cpu-and-memory)

### 最低要求

建议根据 {% data variables.product.product_location %} 的用户许可数选择不同的硬件配置。 如果预配的资源超过最低要求，您的实例将表现出更好的性能和扩展。

{% data reusables.enterprise_installation.hardware-rec-table %}

### 存储器

我们建议为 {% data variables.product.prodname_ghe_server %} 配置具有高每秒输入/输出操作数 (IOPS) 和低延迟的高性能 SSD。 工作负载是 I/O 密集型的。 如果使用裸机管理程序，建议直接连接磁盘或使用存储区域网络 (SAN) 中的磁盘。

您的实例需要一个独立于根磁盘的持久数据磁盘。 更多信息请参阅“[系统概述](/enterprise/admin/guides/installation/system-overview)”。

{% ifversion ghes %}

To configure {% data variables.product.prodname_actions %}, you must provide external blob storage. 更多信息请参阅“[{% data variables.product.prodname_ghe_server %} 的 {% data variables.product.prodname_actions %} 使用入门](/admin/github-actions/getting-started-with-github-actions-for-github-enterprise-server##external-storage-requirements)”。

{% endif %}

The available space on the root filesystem will be 50% of the total disk size. 您可以通过构建一个新实例或使用现有实例来调整实例的根磁盘大小。 For more information, see "[System overview](/enterprise/admin/guides/installation/system-overview#storage-architecture)" and "[Increasing storage capacity](/enterprise/{{ currentVersion }}/admin/guides/installation/increasing-storage-capacity)."

### CPU 和内存

{% data variables.product.prodname_ghe_server %} 需要的 CPU 和内存资源取决于用户的活动水平、自动化和集成。

{% ifversion ghes %}

If you plan to enable {% data variables.product.prodname_actions %} for the users of your {% data variables.product.prodname_ghe_server %} instance, you may need to provision additional CPU and memory resources for your instance. 更多信息请参阅“[{% data variables.product.prodname_ghe_server %} 的 {% data variables.product.prodname_actions %} 使用入门](/admin/github-actions/getting-started-with-github-actions-for-github-enterprise-server#review-hardware-considerations)”。

{% endif %}

{% data reusables.enterprise_installation.increasing-cpus-req %}

{% warning %}

**警告：** 我们建议用户配置 web 挂钩事件来通知外部系统有关 {% data variables.product.prodname_ghe_server %} 上的活动。 自动检查更改或 _轮询_将对实例的性能和可扩展性产生不利影响。 更多信息请参阅“[关于 web 挂钩](/github/extending-github/about-webhooks)”。

{% endwarning %}

有关监控 {% data variables.product.prodname_ghe_server %} 容量和性能的更多信息，请参阅“[监控您的设备](/admin/enterprise-management/monitoring-your-appliance)”。

您可以增加实例的 CPU 或内存资源。 更多信息请参阅“[增加 CPU 或内存资源](/enterprise/admin/installation/increasing-cpu-or-memory-resources)”。
