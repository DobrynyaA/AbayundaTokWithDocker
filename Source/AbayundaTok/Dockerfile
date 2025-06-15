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
RUN dotnet dev-certs https --clean

RUN apt-get update && \
    apt-get install -y ffmpeg && \
    apt-get clean

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

COPY ["AbayundaTok.DAL/AbayundaTok.DAL.csproj", "AbayundaTok.DAL/"]
COPY ["AbayundaTok.BLL/AbayundaTok.BLL.csproj", "AbayundaTok.BLL/"]
COPY ["AbayundaTok.PresentationLayer/AbayundaTok.PresentationLayer.csproj", "AbayundaTok.PresentationLayer/"]
COPY ["AbayundaTok/AbayundaTok.csproj", "AbayundaTok/"]
RUN dotnet restore "AbayundaTok/AbayundaTok.csproj"

COPY . .

WORKDIR "/src/AbayundaTok"
RUN dotnet build "AbayundaTok.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "AbayundaTok.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "AbayundaTok.dll"]