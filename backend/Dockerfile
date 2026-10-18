FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

COPY ["global.json", "./"]
COPY ["src/School.Api/School.Api.csproj", "src/School.Api/"]
COPY ["src/School.Application/School.Application.csproj", "src/School.Application/"]
COPY ["src/School.Domain/School.Domain.csproj", "src/School.Domain/"]
COPY ["src/School.Infrastructure/School.Infrastructure.csproj", "src/School.Infrastructure/"]
RUN dotnet restore "src/School.Api/School.Api.csproj"

COPY . .
RUN dotnet publish "src/School.Api/School.Api.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "School.Api.dll"]
