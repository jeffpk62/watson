---

copyright:
  years: 2015, 2019
lastupdated: "2019-03-08"

keywords: Watson tokens,Cloud Foundry

subcollection: watson

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:deprecated: .deprecated}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Watson 令牌
{: #gs-tokens-watson-tokens}

使用 {{site.data.keyword.watson}} 令牌来编写向 {{site.data.keyword.ibmwatson}} 服务发出经认证的请求的应用程序，而无需在每个调用中嵌入 Cloud Foundry 服务凭证。
您必须对服务使用 Cloud Foundry 服务凭证来获取 {{site.data.keyword.watson}} 令牌。有关更多信息，请参阅[使用 Cloud Foundry 服务凭证进行认证](/docs/services/watson?topic=watson-creating-credentials)。您获取特定服务实例的令牌，此令牌仅适用于该实例。
{: shortdesc}

缺省情况下，{{site.data.keyword.cloud_notm}} 对所有新的服务实例使用 Identity and Access Management (IAM) 认证。此处描述的令牌适用于 Cloud Foundry 服务凭证。有关 IAM 认证的更多信息，请参阅[使用 IAM 令牌进行认证](/docs/services/watson?topic=watson-iam#iam)。
{: important}

您可以在 {{site.data.keyword.cloud}} 中编写认证代理，此代理获取令牌并返回给客户机应用程序，然后客户机应用程序可使用令牌来直接调用服务。使用此代理，无需通过中间服务器端应用程序传输所有服务请求，否则需要避免公开来自客户机应用程序的服务凭证。

## 获取令牌
{: #gs-tokens-obtain}

要获取服务的令牌，向 `authorization` API 的 `token` 方法发送 HTTP `GET` 请求。主机依赖于您所使用的位置和服务。您可以从服务的 API 的端点标识主机。

要标识想要获取其令牌的服务，通过方法的 `url` 查询参数传递服务的基本 URL。例如，以下 curl 命令使用 `url` 参数访存 {{site.data.keyword.texttospeechshort}} 服务的令牌：

```bash
url=https://stream.watsonplatform.net/text-to-speech/api
```
{: codeblock}

将服务的 HTTP 基本认证凭证与请求一起传递，就如同不使用令牌的任何调用一样。例如，以下 curl 命令获取 {{site.data.keyword.texttospeechshort}} 服务的令牌：

```bash
curl -X GET --user {username}:{password} \
"https://stream.watsonplatform.net/authorization/api/v1/token?url=https://stream.watsonplatform.net/text-to-speech/api"
```
{: pre}

`--user` 选项传递服务的*用户名*和*密码*的并置内容。您可从 {{site.data.keyword.cloud_notm}} 或从绑定到服务实例的应用程序的 `VCAP_SERVICES` 环境变量获取这些凭证。确保使用服务凭证，而不是 {{site.data.keyword.cloud_notm}} 登录标识和密码。

此方法通过已加密的有效内容的单千字节 Base64 编码形式返回令牌。

令牌具有一小时的生存时间 (TTL)，在此时间后将无法再使用相应令牌建立与服务的连接。已使用此令牌建立的现有连接不受超时影响。尝试传递已到期或无效的令牌会从 {{site.data.keyword.cloud_notm}} 产生 HTTP `401 Unauthorized` 状态码。需要准备应用程序代码以刷新令牌，从而响应此返回码。

## 使用令牌
{: #gs-tokens-watson-use}

使用 `X-Watson-Authorization-Token` 请求头、`watson-token` 查询参数或 cookie 将令牌传递给服务的 HTTP 接口。必须随每个 HTTP 请求一起传递令牌。

以下 curl 示例演示前两个方法。

1.  获取 {{site.data.keyword.texttospeechshort}} 服务的令牌，并将其写入名为 `token` 的文件。

    ```bash
    curl -X GET --user {username}:{password} \
    --output token \
    "https://stream.watsonplatform.net/authorization/api/v1/token?url=https://stream.watsonplatform.net/text-to-speech/api"
```
    {: pre}

1.  将调用中 `token` 文件的内容传递到 {{site.data.keyword.texttospeechshort}} API 的 `synthesize` 方法。第一个命令通过 `X-Watson-Authorization-Token` 头传递令牌。第二个命令通过 `watson-token` 查询参数传递。这两个命令是等效的。每个命令都将调用的输出写入到名为 `hello_world.wav` 的文件。

    -   *从 UNIX 或 Linux shell，*将 `token` 文件的内容重定向到命令：

        ```bash
      curl -X POST \
      --header "X-Watson-Authorization-Token: $(< ./token)" \
      --header "Content-Type: application/json" \
      --header "Accept: audio/wav" \
      --data "{\"text\":\"hello world\"}" \
      --output hello_world.wav \
      "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize"
      ```
        {: pre}

        ```bash
      curl -X POST \
      --header "Content-Type: application/json" \
      --header "Accept: audio/wav" \
      --data "{\"text\":\"hello world\"}" \
      --output hello_world.wav \
      "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize?watson-token=$(< ./token)"
      ```
        {: pre}

    -   *从 Windows 命令提示符，*在命令中嵌入 `token` 文件的内容：

        ```bash
        curl -X POST
        --header "X-Watson-Authorization-Token: {token_string}"
        --header "Content-Type: application/json"
        --header "Accept: audio/wav"
        --data "{\"text\":\"hello world\"}"
        --output hello_world.wav
        "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize"
        ```
        {: pre}

        ```bash
        curl -X POST
        --header "Content-Type: application/json"
        --header "Accept: audio/wav"
        --data "{\"text\":\"hello world\"}"
        --output hello_world.wav
        "https://stream.watsonplatform.net/text-to-speech/api/v1/synthesize?watson-token={token_string}"
        ```
        {: pre}
