# Dockerfile - WSO2 Micro Integrator simple (Windows/Docker Desktop compatible)

# Forcer l'architecture amd64 pour éviter l'erreur libtinfo
FROM wso2/wso2mi:4.4.0


# Copier le CAR file si présent
# ⚠️ Assurez-vous que le build Maven a généré le CAR dans target/
COPY target/billingintegration_1.0.0.car /home/wso2carbon/wso2mi-4.4.0/repository/deployment/server/carbonapps/

# Exposer les ports WSO2
EXPOSE 8290 8253 9164

# Définir le répertoire de travail
WORKDIR /home/wso2carbon/wso2mi

# Démarrer le serveur WSO2 MI
CMD ["/home/wso2carbon/wso2mi/bin/micro-integrator.sh", "-b", "0.0.0.0"]
