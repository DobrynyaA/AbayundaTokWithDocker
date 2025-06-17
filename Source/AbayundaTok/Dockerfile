#See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

# Настройка переменных окружения для HTTP
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Development
ENV ASPNETCORE_HTTP_PORT=8080
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false
ENV ASPNETCORE_Kestrel__Endpoints__Http__Url=http://+:8080
ENV ASPNETCORE_Kestrel__Endpoints__Http__Protocols=Http1AndHttp2
ENV ASPNETCORE_Kestrel__Endpoints__Http__UseHttps=false

# Отключаем HTTPS
#RUN dotnet dev-certs https --clean

RUN apt-get update && \
    apt-get install -y ffmpeg && \
    apt-get clean

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Настройка NuGet
ENV DOTNET_SYSTEM_NET_HTTP_USESOCKETSHTTPHANDLER=0
ENV DOTNET_SYSTEM_NET_HTTP_SOCKETSHTTPHANDLER_ENABLEMULTIPLECONNECT=0
ENV NUGET_CREDENTIALPROVIDER_TIMEOUT=0
ENV DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1
ENV NUGET_XMLDOC_MODE=skip
ENV DOTNET_NOLOGO=1
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1
ENV NUGET_HTTP_TIMEOUT=300
ENV NUGET_PACKAGES=/root/.nuget/packages

RUN mkdir -p /root/.nuget/NuGet/ && \
    echo '<?xml version="1.0" encoding="utf-8"?>' > /root/.nuget/NuGet/NuGet.Config && \
    echo '<configuration>' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '  <packageSources>' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '    <clear />' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '    <add key="nuget-v2" value="https://www.nuget.org/api/v2/" />' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '  </packageSources>' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '  <packageRestore>' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '    <add key="enabled" value="True" />' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '    <add key="automatic" value="True" />' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '  </packageRestore>' >> /root/.nuget/NuGet/NuGet.Config && \
    echo '</configuration>' >> /root/.nuget/NuGet/NuGet.Config

COPY ["AbayundaTok.DAL/AbayundaTok.DAL.csproj", "AbayundaTok.DAL/"]
COPY ["AbayundaTok.BLL/AbayundaTok.BLL.csproj", "AbayundaTok.BLL/"]
COPY ["AbayundaTok.PresentationLayer/AbayundaTok.PresentationLayer.csproj", "AbayundaTok.PresentationLayer/"]
COPY ["AbayundaTok/AbayundaTok.csproj", "AbayundaTok/"]

# Восстанавливаем пакеты с дополнительными параметрами
RUN dotnet restore "AbayundaTok/AbayundaTok.csproj" --verbosity detailed --configfile /root/.nuget/NuGet/NuGet.Config --no-cache

COPY . .
WORKDIR "/src/AbayundaTok"
RUN dotnet build "AbayundaTok.csproj" -c ${BUILD_CONFIGURATION} -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "AbayundaTok.csproj" -c ${BUILD_CONFIGURATION} -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "AbayundaTok.dll"]