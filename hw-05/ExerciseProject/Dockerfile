# Stage 1
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
WORKDIR exerciseProject

COPY ./ExerciseProject/*.csproj ./
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o out

# Stage 2
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
WORKDIR /exerciseProject
COPY --from=build /exerciseProject/out .
ENTRYPOINT ["dotnet", "ExerciseProject.dll"]