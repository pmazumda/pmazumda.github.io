---
layout: post
title: How to Customize the Look and Feel of Harbor
date: '2024-01-02T12:47:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2024-01-04
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- Kubernetes
- Helm
- Harbor
- New technology
- Cloud native
readtime: true
share-title: How to Customize the Look and Feel of Harbor.
---
## Customizing Look and Feel of Harbor Registry

Recently while configuring a secondary Harbor instance for our development use, we thought if it is possible to do some customizations on the look and feel of Harbor and we came across the following section in the official Harbor documentation.

[Harbor docs | Customize the Look and Feel of Harbor (goharbor.io)](https://goharbor.io/docs/2.10.0/build-customize-contribute/customize-look-feel/)  

However, the documentation doesn't mention the complete steps on how to achieve this when Harbor is installed using Helm chart.

In this article, I have outlined the steps we implemented to customize few of the attributes.

So, the primary file where all the customization related configurations are stored is *setting.json*, As per the above documentation, this file is present under `$HARBOR_DIR/src/portal/src` folder but this is when you deploy Harbor using the installer. 

In our case, I could see that this file is part of the Harbor portal container. I am using Harbor **v2.6.2** and the file is present under `usr/share/nginx/html`
If you open the file, you'll see the default content is 

    {
      "headerBgColor": {
        "darkMode": "",
        "lightMode": ""
      },
      "loginBgImg": "",
      "loginTitle": "",
      "product": {
        "name": "",
        "logo": "",
        "introduction": ""
      }
    }

- Step 1 

 Change the values of configuration if you want to override the default style to your own. Here are references:

-   **headerBgColor**: The background color of the page header, support either HEX or RGB value. e.g:  `#004a70`  and  `rgb(210,110,235)`.
    -   **darkMode**: The background color of the page header for the dark mode.
    -   **lightMode**: The background color of the page header for the light mode.
-   **loginBgImg**: The name path of the background image displayed in the login page, e.g: ‘login_bg.png’. The image file should be put in the  `images`  folder. Suggest the size of this image should be bigger than 800px*600px.
-   **loginTitle**: The full product title displayed in the login page.
-   **Product**: Contain metadata / description of the product.
    -   **name**: The name of the product.
    -   **logo**: The name path of the product logo, e.g: ‘logo.png’. The image file should be put in the  `images`  folder.
    -   **introductions**: The introduction about the product, which are displayed in the  `About`  dialog.

Once updated, create a folder called as **customize** and save it under the templates folder of the Harbor helm chart using which you plan to update.

 - Step2

Create a configmap named as harborcustom loading the datatype as setting.json  we created in Step 1.

    kubectl create configmap  harborcustom --from-file=setting.json=customize/setting.json --namespace <NAMESPACE>

 - Step3

Mount the created configmap in Step2 as a volume inside the pod.
Add relevant mount section inside the **deployment.yaml** file of the portal manifest.


    volumeMounts:
      - mountPath: /usr/share/nginx/html/setting.json
         name: custom
         subPath: setting.json
    
    volumes:
    - name: custom
      configMap:
         defaultMode: 420
         name: harborcustom       

Save the file and deploy Harbor using the helm chart. Use harbor install or upgrade commands with appropriate flags to perform the upgrade or install if doing from scratch. 

And that's all, now you should be able to see the customized Title, name or logo whichever you modified as per your requirement.

I hope this short and quick article will help up a lot if you want to do some simple customizations on your Harbor, for a detailed description on how to build from Harbor source code, customize deployments, and contribute to the open-source Harbor project, please refer the original documentation.

Thank You!!
