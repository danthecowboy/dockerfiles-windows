# escape=`
FROM microsoft/windowsservercore as msi

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV MONGO_VERSION 3.4.4
ENV MONGO_DOWNLOAD_URL http://downloads.mongodb.org/win32/mongodb-win32-x86_64-2008plus-ssl-${MONGO_VERSION}-signed.msi
ENV MONGO_DOWNLOAD_SHA256 73d1121a1fe58fe29c4fb7b6f4b79a4d33afa666f357960419ef83635396f977

RUN Write-Host ('Downloading {0} ...' -f $env:MONGO_DOWNLOAD_URL)
RUN Invoke-WebRequest -Uri $env:MONGO_DOWNLOAD_URL -OutFile 'mongodb.msi'
RUN Write-Host ('Verifying sha256 ({0}) ...' -f $env:MONGO_DOWNLOAD_SHA256)
RUN if ((Get-FileHash mongodb.msi -Algorithm sha256).Hash -ne $env:MONGO_DOWNLOAD_SHA256) { `
      Write-Host 'FAILED!'; `
      exit 1; `
    }
RUN Start-Process msiexec.exe -ArgumentList '/i', 'mongodb.msi', '/quiet', '/norestart', 'INSTALLLOCATION=C:\mongodb', 'ADDLOCAL=Server,Client,MonitoringTools,ImportExportTools,MiscellaneousTools' -NoNewWindow -Wait
RUN Remove-Item C:\mongodb\bin\*.pdb

FROM microsoft/nanoserver

COPY --from=msi C:\mongodb\ C:\mongodb\
COPY --from=msi C:\windows\system32\msvcp140.dll C:\windows\system32
COPY --from=msi C:\windows\system32\vcruntime140.dll C:\windows\system32

RUN mkdir C:\data\db & setx /m PATH %PATH%;C:\mongodb\bin

VOLUME C:\data\db

EXPOSE 27017

CMD ["mongod.exe"]
