# Utiliser l'image de base .NET SDK pour la construction et l'exécution
FROM mcr.microsoft.com/dotnet/sdk:7.0

# Définir le répertoire de travail
WORKDIR /app

# Copier et restaurer les dépendances
COPY Worker.csproj .
RUN dotnet restore

# Copier le reste de l'application et construire
COPY . .
RUN dotnet publish -c Release -o out

# Définir le répertoire de travail pour l'application publiée
WORKDIR /app/out

# Définir la commande de démarrage
ENTRYPOINT ["dotnet", "Worker.dll"]