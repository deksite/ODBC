# ODBC
ODBC 64Bits ARM

1) Re-extraia o MSI para um diretório limpo
	1.	Crie uma pasta temporária limpa, por exemplo:
        
        md C:\tmp\mysqlodbc_extract


	2.	Extraia o MSI em modo administrativo:
    
        msiexec /a "C:\Downloads\mysql-connector-odbc-9.3.xx-winx64.msi" /qb TARGETDIR="C:\tmp\mysqlodbc_extract"

    3.	Dentro de C:\tmp\mysqlodbc_extract\, você verá ao menos duas sub-pastas:

        •	PFiles      → contém os arquivos 32-bit
	    •	PFiles64    → contém os arquivos 64-bit

2) Copie somente o conteúdo de PFiles64

    Isto é CRUCIAL: use apenas a pasta PFiles64\MySQL\MySQL Connector ODBC 9.3 (a versão x64) e não misture com nada de PFiles.
    No PowerShell (Admin):

    $src = 'C:\tmp\mysqlodbc_extract\PFiles64\MySQL\MySQL Connector ODBC 9.3'
    $dst = 'C:\Program Files\MySQL\MySQL Connector ODBC 9.3'

    Limpa qualquer sobra e recria destino
    Remove-Item $dst -Recurse -Force -ErrorAction SilentlyContinue
    New-Item -ItemType Directory -Path $dst -Force

    Copia tudo (DLLs ANSI/Unicode/Setup, plugin\, libcrypto-3-x64.dll, etc.)
    Copy-Item "$src\*" -Destination $dst -Recurse -Force




    Verifique que em:

    C:\Program Files\MySQL\MySQL Connector ODBC 9.3

    você tem, entre outros:
	•	myodbc9a.dll       (driver ANSI x64)
	•	myodbc9w.dll       (driver Unicode x64)
	•	myodbc9S.dll       (Setup DLL x64!)
	•	plugin\mysql_native_password.dll (e demais plugins)
	•	libcrypto-3-x64.dll, libssl-3-x64.dll, etc.


3) Ajuste o PATH (opcional, mas recomendado)

   Ainda no PowerShell Admin:

   [Environment]::SetEnvironmentVariable(
  'Path',
  $env:Path + ';C:\Program Files\MySQL\MySQL Connector ODBC 9.3',
  'Machine'
    )

    Isso garante que DLLs de criptografia e SASL possam ser encontradas.

4) Reimporte seu .reg (se necessário)

    Use este .reg (ajustado apenas se você mudou o caminho de instalação) para registrar o driver e o DSN “EXEMPLO”:


    Windows Registry Editor Version 5.00

    [HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBCINST.INI\ODBC Drivers]
    "MySQL ODBC 9.3 ANSI Driver"="Installed"
    "MySQL ODBC 9.3 Unicode Driver"="Installed"

    [HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBCINST.INI\MySQL ODBC 9.3 ANSI Driver]
    "Driver"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\myodbc9a.dll"
    "Setup"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\myodbc9S.dll"
    "FileUsage"=dword:00000001

    [HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBCINST.INI\MySQL ODBC 9.3 Unicode Driver]
    "Driver"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\myodbc9w.dll"
    "Setup"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\myodbc9S.dll"
    "FileUsage"=dword:00000001

    [HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBC.INI\ODBC Data Sources]
    "EXEMPLO"="MySQL ODBC 9.3 ANSI Driver"

    [HKEY_LOCAL_MACHINE\SOFTWARE\ODBC\ODBC.INI\EXEMPLO]
    "Driver"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\myodbc9a.dll"
    "Server"="IP"
    "Database"="XXX"
    "UID"="XXX"
    "PWD"="XXXX"
    "Port"="3306"
    "Option"="3"
    "PLUGIN_DIR"="C:\\Program Files\\MySQL\\MySQL Connector ODBC 9.3\\plugin"
    
