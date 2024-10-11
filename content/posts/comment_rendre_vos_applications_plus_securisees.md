---
title: "Developpeurs : Comment rendre vos applications plus sécurisées ?"
date: 2024-10-07T12:03:38+01:00
type: 'post'
categories: ['security']
tags: ['devsecops', 'SDLC', 'security', 'trusted software supply chain']
---
La sécurité est un vaste sujet qui est transverse à toutes les composantes d'une organisation, impactant aussi bien les processus, les technologies que les individus, et nécessitant une approche globale et proactive pour protéger les données et les systèmes contre les menaces internes et externes. Dans ce post, je vais faire un focus sur la sécurité des applications. 

## Qu’est ce qu’on doit considérer et sur quoi on doit porter notre attention ?

Il est bien connu qu'aucun logiciel n'est exempt de bugs. À mesure que les systèmes deviennent plus complexes, certains de ces bugs se transforment inévitablement en vulnérabilités de sécurité. Il est donc essentiel de prendre des mesures pour réduire ces failles et les corriger le plus tôt possible, car le coût de correction d'une vulnérabilité augmente de manière exponentielle lorsqu'elle est découverte après la mise en production. C’est ce qui est représenté en bleu sur ce graphique.

![coût de correction d'une vulnérabilité](/img/2024-10-07/cout_vulnerabilites.png)

Si nous examinons les différentes étapes du développement d'une application et analysons à quel moment la sécurité ou, de manière générale, les vulnérabilités sont introduites, nous constatons que la majorité de ces failles apparaissent souvent dès les premières phases, comme la phase de conception et de codage, avant d'être détectées plus tard dans les phases de test ou, pire encore, après la mise en production.

## Comment évolue le coût ?

Eh bien, il se trouve que le coût fonctionne à peu près comme ceci : si vous prenez un coût de base de correction d'un bug à l'étape de codage, disons 1x, ce coût peut atteindre 640x dans certains cas une fois que la vulnérabilité est en production (courbe en rouge). Il est donc infiniment plus coûteux de corriger une vulnérabilité après son déploiement qu’à un stade précoce du développement. Cela constitue une incitation majeure à bien faire les choses dès le début et à détecter ces failles le plus tôt possible. Cette approche est connue sous le nom de ‘déplacer la sécurité vers la gauche’ ou ‘shift left security’.

Maintenant que nous savons où concentrer notre attention, nous pouvons entrer dans le vif du sujet. Il est essentiel d'introduire la sécurité le plus tôt possible dans le processus de développement pour minimiser les risques. Je vois trois phases sur lesquelles nous pouvons intervenir : le cycle de développement d’une application - SDLC (Software Development Life Cycle), les bonnes pratiques de codage, et les tests de vulnérabilités. En agissant sur ces trois fronts, nous renforçons la sécurité à chaque étape du développement d'une application.

## La sécurité pendant le cycle de développement d’une application

Il existe plusieurs modèles pour structurer le cycle de développement d'une application, chacun ayant ses particularités et ses approches vis-à-vis de la sécurité. Les modèles les plus courants sont le **modèle traditionnel**, le **modèle DevOps**, qui favorise l'intégration continue et la collaboration entre les équipes, et enfin le **modèle DevSecOps**, une évolution de DevOps qui place la sécurité au cœur de chaque phase du développement. Chaque modèle présente des avantages et des défis uniques en ce qui concerne l'intégration de la sécurité, notamment en matière de détection et de correction des vulnérabilités.

Le **modèle traditionnel** suit une approche linéaire, généralement en cascade, où la sécurité est introduite tardivement, juste avant la mise en production. Cette approche séquentielle isole les équipes de développement des équipes d’exploitation, ce qui engendre une faible collaboration. Le problème majeur de ce modèle est que la détection des vulnérabilités arrive souvent trop tard, rendant les correctifs coûteux et difficiles à implémenter.

![Modèle Dev & Ops traditionnel](/img/2024-10-07/devops_linear.png)

Ensuite, le **modèle DevOps** vise à combler le fossé entre développement (Dev) et opérations (Ops). Grâce à une approche cyclique, il permet d'obtenir un feedback continu sur ce qui a été livré, facilitant ainsi l'amélioration des futures versions. Cependant, la sécurité reste souvent traitée comme une réflexion après coup, plutôt qu’une priorité intégrée dès le départ.

![Modèle DevOps](/img/2024-10-07/devops_model.png)

C’est pourquoi le **modèle DevSecOps** a émergé. Il intègre la sécurité au cœur du processus de développement, en l'incorporant dès les premières phases de conception et de codage. Grâce à cette approche, les vulnérabilités peuvent être détectées et corrigées à des stades beaucoup plus précoces, réduisant ainsi considérablement les risques et les coûts. Ce modèle permet également de tirer parti de l'automatisation et des bonnes pratiques de sécurité tout au long du cycle de vie de l’application, garantissant une meilleure résilience face aux menaces.

![Modèle DevSecOps](/img/2024-10-07/devsecops-model-removebg.png)

En résumé, passer d’une approche traditionnelle à du DevSecOps permet non seulement d'améliorer la collaboration et l'efficacité, mais aussi de garantir une sécurité renforcée à chaque étape du développement.

## Les bonnes pratiques de codage

Il est essentiel de s'intéresser aux bonnes pratiques de développement pour assurer la sécurité des applications dès les premières étapes de leur création. En adoptant des standards de codage sécurisé, on limite les vulnérabilités qui pourraient être exploitées plus tard. Par ailleurs, il est tout aussi important de connaître les erreurs communes à éviter. 


![OWASP](/img/2024-10-07/owasp.png)

L'**OWASP** (Open Web Application Security Project) est une excellente ressource en la matière, offrant des recommandations détaillées et une liste des vulnérabilités les plus courantes à surveiller. Le célèbre OWASP Top Ten met en avant les erreurs de sécurité fréquentes, telles que les injections et les failles de validation des entrées, ce qui en fait une référence incontournable pour les développeurs souhaitant renforcer la robustesse de leurs applications.

![Reference architecture](/img/2024-10-07/reference-architecture.png)

Il est également crucial de s'appuyer sur des **architectures de référence** pour garantir la sécurité et la fiabilité des applications. Ces architectures fournissent des modèles éprouvés, permettant de structurer les applications de manière optimale tout en intégrant les bonnes pratiques de sécurité. En suivant ces modèles, les développeurs et architectes peuvent éviter des erreurs fréquentes et s'assurer que les systèmes sont construits sur des bases solides. Red Hat dispose d’un site web dédié où sont listées différentes architectures de référence (Red Hat Architecture Center). Ces ressources aident à standardiser les pratiques au sein des équipes et à garantir une approche cohérente et sécurisée du cycle de vie des applications.

Les développeurs ne réécrivent pas tout leur code à partir de zéro, mais s'appuient souvent sur des bibliothèques tierces, qu'elles soient open source ou propriétaires. Toutefois, il est important de s'assurer que ces bibliothèques sont fiables et exemptes de vulnérabilités. Même des bibliothèques populaires et bien établies peuvent contenir des failles, comme l’a démontré la célèbre **faille Log4j**, qui a touché un grand nombre de systèmes. Il est donc impératif de choisir soigneusement les bibliothèques utilisées, de vérifier régulièrement leur sécurité, et de suivre de près les mises à jour et les correctifs de sécurité. Travailler avec des bibliothèques de confiance réduit considérablement les risques d'exposer une application à des attaques.

![SBOM](/img/2024-10-07/sbom.png)

Dans ce même contexte, il est crucial de savoir exactement ce qui compose une application. Le **Software Bill of Materials (SBOM)** est une pratique qui permet aux équipes de développement de maîtriser l'ensemble des composants, bibliothèques et dépendances utilisés dans leurs applications, comme on le ferait pour une chaîne d'approvisionnement. À travers un SBOM, les équipes peuvent tracer chaque élément de l’application, connaître son origine, sa version, ainsi que les relations entre les différents composants. Cette traçabilité est cruciale pour identifier rapidement les failles potentielles et réagir efficacement en cas de vulnérabilité, comme ce fut le cas avec la faille Log4j. En ayant une vue complète de leur chaîne logicielle, les équipes sont en mesure de suivre l'évolution de chaque composant et de garantir une mise à jour proactive et sécurisée, renforçant ainsi la résilience de l'application face aux attaques.

## Les tests de vulnérabilités

![SCA SAST DAST](/img/2024-10-07/sca-sast-dast.png)

Pour rappel, j’ai mentionné que le principe du DevSecOps est de s’appuyer sur des outils d’automatisation pour identifier les vulnérabilités. Trois outils majeurs dans ce domaine sont le **Static Application Security Testing** (SAST), le **Dynamic Application Security Testing** (DAST) et le **Software Composition Analysis** (SCA).

Le **SAST** est une méthode d'analyse de la sécurité des applications qui examine le code source sans l'exécuter. En d'autres termes, il scrute le code en profondeur à la recherche de vulnérabilités potentielles, telles que les erreurs de programmation ou les failles de sécurité, dès que le code est écrit. Cette approche est dite "statique" car elle analyse le code à un stade précoce du développement, ce qui permet d’identifier et de corriger les failles avant même que le logiciel ne soit déployé. L’utilisation de SAST aide ainsi à intégrer la sécurité dès la phase de codage, réduisant les coûts de correction des vulnérabilités en amont.

Le **DAST**, en revanche, permet aux équipes d'analyser les applications en cours d'exécution sans avoir besoin de connaître la conception interne de l’application ni d’avoir accès au code source. Contrairement au SAST, le DAST est utilisé après le déploiement de l'application dans un environnement de test ou en production. Il détecte des problèmes tels que les failles de sécurité dans la configuration de l'application ou les vulnérabilités connues (www.cvedetails.com), offrant ainsi une vue "dynamique" de la sécurité de l'application. 

Le **SCA** est un outil qui analyse les bibliothèques et composants tiers utilisés dans une application pour détecter des vulnérabilités connues et des problèmes de licence. Le SCA se concentre sur les composants externes, assurant ainsi que l'application soit sécurisée à tous les niveaux, y compris les dépendances externes.

## Comment Red Hat accompagne ses clients pour améliorer la sécurité des applications ?

![Red Hat security framework](/img/2024-10-07/redhat-security.png)

Red Hat, acteur majeur du monde open source, propose une suite complète de solutions et de pratiques pour accompagner ses clients à sécuriser leurs chaînes de développement et de déploiement d’applications. Dans cet article je vais parler de la famille de produit **Red Hat Trusted Software Supply Chain** (RH TSSC) 

Il est essentiel de comprendre que l'objectif principal n'est pas simplement de détecter des bugs, mais de réduire leur nombre dès le départ. Cela s'avère bien plus efficace et économique que de consacrer des ressources à la correction de bugs après coup. 

**1 - Déplacer la sécurité vers la gauche**

![Shift left security](/img/2024-10-07/shift-left-security.png)

La première chose qui vient à l’esprit c’est de rajouter une couche de sécurité lors du build. Sauf que quand on se réfère au cycle de développement, la première phase est le codage. Et c’est sur cette phase qu’on doit mettre le paquet.

**Red Hat Dependency Analytics** (composant de **Red Hat Trusted Profile Analyzer**) aide les développeurs à repérer les vulnérabilités dans les bibliothèques et les dépendances de leur application depuis leurs IDE. Elle fournit des alertes en temps réel sur les failles de sécurité, permettant aux développeurs de les corriger rapidement et de sécuriser leur code dès la phase de développement.

**Red Hat Advanced Cluster Security** propose un moteur d’analyse des vulnérabilités (SCA) qui peut être intégré directement dans le cycle de développement. Grâce à une approche DevSecOps, ce moteur peut être utilisé via des interfaces en ligne de commande (CLI) et fournir des rapports détaillés des vulnérabilités. Ces rapports permettent aux équipes de développement d'identifier précisément les failles de sécurité présentes dans le code ou les dépendances utilisées, et d'agir rapidement pour les corriger avant même que l'application ne soit déployée. 

**2 -  Assurer l'intégrité logicielle à l'image de la chaîne du froid**

![Shift left security](/img/2024-10-07/integrity.png)

L'allusion à la chaîne du froid pour la trusted software supply chain repose sur une analogie entre la manière dont on garantit l'intégrité des produits périssables et celle dont on protège l'intégrité des logiciels tout au long de leur cycle de vie. Voici pourquoi cette comparaison est pertinente :

**Préservation de l'intégrité** : Dans la chaîne du froid, la température doit être maintenue de manière rigoureuse à chaque étape pour éviter la détérioration des produits. De la même manière, dans une TSSC, l'intégrité des logiciels (code, dépendances, artefacts) doit être assurée à chaque phase, de la création à la distribution, afin de prévenir les risques de compromission ou d’altération malveillante.

**Contrôle à chaque étape** : Tout comme dans la chaîne du froid où chaque étape — production, transport, stockage — est surveillée pour garantir que la température reste adéquate, la chaîne d'approvisionnement logicielle doit être sécurisée à chaque phase. Chaque point de passage présente des risques qui doivent être contrôlés pour éviter les vulnérabilités ou l'injection de code malveillant.

**Traçabilité** : Dans la chaîne du froid, il est crucial de tracer l’origine et le parcours des produits pour identifier d’éventuelles ruptures de température. De la même manière, dans une chaîne d'approvisionnement logicielle, la traçabilité est fondamentale pour savoir d'où viennent les composants, qui les a modifiés et quand, ce qui permet de détecter rapidement les anomalies et d’assurer la transparence.

Red Hat propose **Trusted Artifact Signer** pour garantir l'intégrité des artefacts logiciels dans la chaîne d'approvisionnement. Cette solution permet de signer les artefacts avec des identités vérifiées, assurant qu'ils proviennent de sources fiables et n'ont pas été altérés. Elle offre une traçabilité complète des composants, en enregistrant et horodatant chaque artefact pour un suivi rigoureux de son parcours, de la création au déploiement. 
De plus, intégré aux pipelines CI/CD, il automatise le processus de signature et de validation, renforçant ainsi la sécurité à chaque étape du développement. 

**3 -Garder le contrôle des composants et dépendances utilisés dans les applications**

![Intégrité des applications](/img/2024-10-07/keep-control.png)

En gardant cette même analogie, dans la chaîne du froid, un manquement à une étape peut rendre les produits dangereux pour la consommation. De même, dans une chaîne logicielle, une faille à un point critique peut entraîner des vulnérabilités exploitables par des cyberattaques. 

Certaines vulnérabilités ne voient le jour qu’après mise en production de l'artefact. Il est donc primordial d'être en capacité de déterminer quelles sont les applications impactées par cette faille.  C’est exactement ce que propose Red Hat avec **Trusted Profile Analyzer** (RHTPA). 

Cet outil permet de centraliser et de tracer toutes les dépendances et composants utilisés dans chaque application, offrant une visibilité complète sur la chaîne d'approvisionnement logicielle. Lorsqu'une nouvelle vulnérabilité est découverte, RHTPA permet d'identifier rapidement les artefacts et applications concernés, facilitant ainsi une réponse rapide et ciblée pour corriger ou mettre à jour les composants vulnérables.

**Le mot de la fin !**

Vous l'avez bien compris, la sécurité est l'affaire de tous. Elle doit être prise en compte dès les premières étapes du développement des applications pour garantir une protection optimale tout au long de leur cycle de vie. Les projets communautaires jouent un rôle clé en tant que précurseurs dans cette démarche. Ils sont souvent les premiers à adopter des pratiques innovantes en matière de sécurité, comme la signature des artefacts. 
Et n’oubliez pas que, dans cet article, on a vu uniquement la sécurité des applications ! 
