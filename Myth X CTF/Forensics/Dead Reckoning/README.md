# Write-Up : Dead Reckoning

Plateforme : MYTHX: AN ENDGAME PROTOCOL

Catégorie : Forensics

Difficulté : Hard
Objectif

L'objectif est d'analyser les traces laissées par un "insider threat" (menace interne) sur le poste de travail WS-019 afin de récupérer un actif sensible exfiltré juste avant la révocation de ses accès.

<p align="center">
<img src="1.png" width="700">
</p>
<p align="center"><em>Interface du challenge présentant l'énoncé et les fichiers</em> </p>
Analyse Initiale

Le challenge fournit un fichier dead_reckoning_vGw5mCk.zip contenant plusieurs artefacts forensic :

    traffic.pcapng : Capture réseau du poste de travail.

    commits.bundle : Historique Git local du développeur.

    README.txt : Contient un message cryptique laissé par le suspect.

    Indice critique : "Dead reckoning. Four bearings. One destination.".

Analyse Forensic Réseau (PCAP)

L'analyse de traffic.pcapng avec Wireshark révèle une activité suspecte via le protocole DNS. Le suspect utilise le domaine interne tunnel.nullroute.internal pour exfiltrer des données par morceaux de 8 caractères hexadécimaux.

<p align="center">
<img src="image_fdc8e6.png" width="700">
</p>
<p align="center"><em>Extraction des segments DNS exfiltrés</em> </p>

En utilisant le filtre dns.qry.name contains "tunnel.nullroute.internal", nous avons identifié les segments suivants :
Paquet	Hexadécimal	ASCII (From Hex)	Rôle
1	45524f58	EROX	Bearing 1
4	30302d5a	00-Z	Bearing 2
9	5458452d	TXE-	Bearing 3
16	46464c4c	FFLL	Bearing 4
25	494f4e30	ION0	Segment additionnel
36	53455353	SESS	Marqueur Session
49	494e522d	INR-	Destination
Analyse de l'Historique Git

Le message "One destination" et l'indice "commit 3" trouvé dans les fichiers suggèrent que l'identifiant final se trouve dans le dépôt.

Commande utilisée :
Bash

git clone commits.bundle repo_challenge
git log -p

L'historique révèle des fautes de frappe délibérées dans les messages de commit formant le mot secret OKAYPN (o-k-a-y-p-n). De plus, le commit 3 (hash 1057bbcc...) est désigné comme le point de contrôle de l'intégration.

<p align="center">
<img src="image_fe3d63.png" width="500">
</p>
<p align="center"><em>Tentative de décodage du hash du commit 3 sur CyberChef</em> </p>
Synthèse du Flag

Le format du flag est MYTHX{<FULL_SESSION_TOKEN>_<KEY_SEGMENT>}.

    KEY_SEGMENT : Basé sur les "Four Bearings" DNS → EROX00-ZTXE-FFLL.

    FULL_SESSION_TOKEN : Basé sur la "Destination" (Hash du commit 3 et segments exfiltrés).

Échecs et Itérations

Lors de la résolution, plusieurs combinaisons ont été testées sans succès, notamment l'intégration du code OKAYPN ou l'inversion des segments DNS. Le niveau "Hard" du challenge suggère que le token final est un assemblage précis entre les données Git et les segments DNS post-SESS.
Conclusion

Ce challenge illustre une exfiltration complexe combinant du DNS Tunneling pour la transmission de données et la stéganographie dans les métadonnées Git pour les identifiants de session. La difficulté résidait dans l'interprétation correcte de l'indice de navigation à l'estime (Dead Reckoning) pour ordonner les segments trouvés.
