* 通过运行以下命令来信任 HTTPS 开发证书：

    ```dotnetcli
    dotnet dev-certs https --trust
    ```

* 上面的命令显示以下输出：

    ```console
    Trusting the HTTPS development certificate was requested. If the certificate 
    is not already trusted we will run the following command:
    'sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain 
    <<certificate>>'
    This command might prompt you for your password to install the certificate on the 
    system keychain.
    The HTTPS developer certificate was generated successfully.
    ```

* 在系统提示时输入管理员用户名和密码。  此时会安装并信任证书。

    有关详细信息，请参阅[信任 ASP.NET Core HTTPS 开发证书](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)。