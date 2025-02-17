* 藉由執行下列命令來信任 HTTPS 開發憑證：

    ```dotnetcli
    dotnet dev-certs https --trust
    ```

* 上述命令會顯示以下輸出：

    ```console
    Trusting the HTTPS development certificate was requested. If the certificate 
    is not already trusted we will run the following command:
    'sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain 
    <<certificate>>'
    This command might prompt you for your password to install the certificate on the 
    system keychain.
    The HTTPS developer certificate was generated successfully.
    ```

* 出現提示時，請輸入系統管理員使用者名稱和密碼。  該憑證現在將會被安裝且受到信任。

    如需詳細資訊，請參閱[信任 ASP.NET Core HTTPS 開發憑證](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)。