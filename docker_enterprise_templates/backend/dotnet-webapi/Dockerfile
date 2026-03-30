# syntax=docker/dockerfile:1.7
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG PROJECT_PATH=src/App/App.csproj
WORKDIR /src
COPY . .
RUN dotnet restore ${PROJECT_PATH}
RUN dotnet publish ${PROJECT_PATH} -c Release -o /app/publish /p:UseAppHost=false

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app
ENV ASPNETCORE_URLS=http://+:8080 \
    DOTNET_EnableDiagnostics=0
COPY --from=build /app/publish .
USER app
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/health || exit 1
# Set APP_DLL to your published dll name, for example MyCompany.Api.dll
ENV APP_DLL=App.dll
ENTRYPOINT ["sh", "-c", "dotnet ${APP_DLL}"]
