---
layout: post
title: Securing access to internal resources using an Identity-Aware Proxy
image: /images/pomerium-logo.jpeg
categories: [Pomerium, Azure, Containers, Docker]
---

Lately I have been looking more into containers, both from the Azure side (AKS) and in my own lab at home (Kubernetes, Docker, containerd etc).

As a part of the home lab, I have set up a few containers for testing and discovering new things. It´s much easier and faster to spin up a container to test something, without the hassle of installing a VM with an OS first.

I have some containers in my lab running a few web services, and they work fine internally. However, sometimes it would be nice to be able to access them externally. But from a security perspective, not all of them have proper authentication mechanisms. In addition, exposing them directly on the internet is not something I would like to do. VPN is something I would try to avoid, since this is a bit of hassle to both set up on different devices and from a usage perspective.

I looked around for a possible solution, and there are a few out there. I would like to use some form of integration with Azure AD (using my credentials from there including MFA) and the best alternative I found, was [Pomerium](https://pomerium.io). This is an Identity-Aware Proxy (IAP), similiar to Google´s BeyondCorp. Pomerium supports different authentication providers, like Google, GitHub and Azure AD.

Overview
![](/images/Pomerium.png)

This provides a secure access to my internal webservices using Azure AD authentication. Even for services with less secure or no authentication at all will this work. Avoiding VPN is a bonus, and can be done using an IAP.

Pomerium provides a Docker image that can be used, along with detailed description for configuring it in your environment. See [this](https://www.pomerium.io/docs/) link for more information.

In my case, I use Nginx as the reverse proxy, with Let´s Encrypt. All my subdomains for services are defined there. For the services using Pomerium, the forward goes to the Pomerium container. So no direct access from Nginx to any of the services containers. 

A policy file needs to be created, then encoded into base64. A simple example for two services:

```yaml
# policy.yaml
- from: https://app1.external.domain
  to: http://app1containername:port
  allowed_users:
    - user1@azuread.domain
- from: https://app2.external.domain
  to: http://app2containername
  allowed_users:
    - user2@azuread.domain
```
Where:
**from** contains the FQDN of the service 
**to** contains the internal container name, with an optional port
**allowed_users** are the user/users you want to have access, and that are present in your Azure AD domain.

If any changes are made, the policy file needs to be base64 encoded again, and the Pomerium container policy environment variable needs to be updated.

In an example, I will show how this looks for accessing [VS Code Server](https://docs.linuxserver.io/images/docker-code-server), which I use to write different types of code.

- Add a new record for my Code Server at my DNS provider
- Set up Nginx for the Code Server FQDN, redirecting to Pomerium IAP
- Request and install a certificate from Let's Encrypt for the new FQDN
- Configure and encode the policy file to reflect the Code Server service, and add it to the compose file
- Add a link in the Pomerium container for the Code Server
- In addition I remove the builtin authentication from Code Server by deleting the HASHED_PASSWORD variable in the compose file
- Start all containers

I access my Code Server FQDN from the internet, and immediately get redirected to login.microsoftonline.com:

![](/images/Pomerium-02.png)

I enter my Azure AD UPN:

![](/images/Pomerium-03.png)

Then my password:

![](/images/Pomerium-04.png)

Since I have activated MFA on my account, I get the challenge:

![](/images/Pomerium-05.png)

Once I have approved the sign-in, I get redirected to the Code Server service.

![](/images/Pomerium-06.png)

Now I have secure access to my Code Server, and can use it from anywhere.

Using an IAP is a way to ensure secure access to services, including services with little or no builtin authentication mechanisms.