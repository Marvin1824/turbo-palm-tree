1. 导航到自托管运行器注册的位置：
   * **在组织或仓库中**，导航到主页并单击 {% octicon "gear" aria-label="The Settings gear" %} **Settings（设置）**。
   * {% ifversion fpt or ghec %}**如果使用企业帐户**：通过访问 `https://github.com/enterprises/ENTERPRISE-NAME`（将 `ENTERPRISE-NAME` 替换为您的企业帐户名称）导航到您的企业帐户。{% elsif ghes or ghae %}**如果使用企业级运行器**：

     1. 在任何页面的右上角，单击 {% octicon "rocket" aria-label="The rocket ship" %}。
     1. 在左边栏中，单击 **Enterprise overview（企业概览）**。
     1. {% endif %} 在企业边栏中，单击 {% octicon "law" aria-label="The law icon" %} **Policies（政策）**。
1. 导航到 {% data variables.product.prodname_actions %} 设置：
   * **在组织或仓库中**：点击左侧栏中的 **Actions**{% ifversion fpt or ghes > 3.1 or ghae-next or ghec %}，然后点击 **Runners（运行器）**{% endif %}。
   * {% ifversion fpt or ghec %}**如果使用企业帐户**{% elsif ghes or ghae %}**如果使用企业级运行器**{% endif %}：在“{% octicon "law" aria-label="The law icon" %} Policies（政策）”下单击 **Actions（操作）**{% ifversion fpt or ghes > 3.1 or ghae-next or ghec %}，然后单击 **Runners（运行器）**选项卡{% endif %}。
