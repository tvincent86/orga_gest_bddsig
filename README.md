# GéoTonyTux

# Organisation et principe de fonctionnement de la base de données SIG


![picto](/blason_na.png)


## Les rôles de connexion et privilèges des groupes
  
  * **Généralité** :

La propriété des bases revient à la DSI - Infrastructure qui créé et ajoute les droits nécessaires.
Le rôle postgres reste le superutilisateur et peut modifier la structure même si il n'est pas le propriétaire.

La propriété des schémas, tables, ... a été transféré au groupe **pre-sig** pour qu'il puisse modifier la structure. Les autres groupes ne pourront pas le faire.  

**ATTENTION :** 
- Restricition d'accès :
  - Le serveur postgreSQL n'est accessible que par les personnes ayant l'autorisation de la DSI - Infrastructure.
  - Le rôle **pre-sig-ro** appartenant au groupe **pre-sig** à les droits de création/modification sur les schémas, tables, fonctions, triggers, ... .


**==> Pour accéder aux données, il faut utiliser d'autres moyens (carte dynamique, flux de données, ...).**

  
  * **Tableaux de répartition** :

  - Base "pre-sig"

|Rôle de connexion|Superutilisateur|Propriétaire des objets|Appartient au groupe|Privilèges sur la structure|Privilèges sur les données|
|:-:|:-:|:-:|:-:|:-:|:-:|
|postgres|x|(par défaut)|-|all|all|
|pre-sig-ro||x|pre-sig|all|all|

  - Base "pre-interfacesig"

|Rôle de connexion|Superutilisateur|Propriétaire des objets|Appartient au groupe|Privilèges sur la structure|Privilèges sur les données|
|:-:|:-:|:-:|:-:|:-:|:-:|
|postgres|x|(par défaut)|-|all|all|
|pre-sig-ro||x|pre-sig|all|all|
|pre-interfacesig-ro||x|pre-interfacesig|aucun|select|

## Règles de dénomination des objets de la base de données

L'ensemble des libellés (schéma, table, champ, vue, ...) doit être écrit en minuscule ce qui permet d'éviter l’utilisation des "" dans les requêtes sql).

## Commentaires sur les objets

L'ensemble des objets (schéma, table, attribut, vue, trigger, ...) contenu dans la base de données doit être commenté comme suit :
- un schéma : description succinte du contenu et de l'usage générique des données
- une table : description succinte du contenu, de l'usage et des particularités si besoin
- un attribut : libellé complet et description succinte si besoin
- une séquence : description de l'usage, de la table et de l'attribut cible
- un trigger / une fonction / une règle : description succinte de son fonctionnement
- une vue : description succinte de son contenu et de son usage

Les contraintes sur les attributs ainsi que les indexes n'ont pas d'obligation de commentaires.

Exemple :

|Objet|Intitulé|Exemple de commentaires |
|:-:|:-:|:-:|
|schéma|`met_cul`|Données métiers sur le théme de la culture|
|table|`m_cul_spect_vivant_struct_p`|Contient les structures de Nouvelle-Aquitaine intervenant dans le spectacle vivant (Point)|
|attribut|`siret`|Code SIRET de la structure|
|vue|`m_cul_v_spect_vivant_struct_detail_p`|Vue contenant les données relatives à la structure avec les détails (classification, catégorie, ...)|
|trigger|`trigger_t1_geo_v_pei_ctr`|Trigger de vue s'exécutant pour une instance d'insertion, de mise à jour ou de suppression |
|fonction |`fct_m_geo_v_pei_ctr`|Fonction métier liée au trigger `t_t1_geo_v_pei_ctr`, gérant les particularités liées à la gestion des données en cas d'insertion, de mise à jour ou de suppression|
|séquence |`m_cul_spect_vivant_struct_p_id_seq`|Séquence dépendante à la table `m_cul_spect_vivant_struct_p` pour l'attribut `id`|


**ATTENTION : pour les fonctions liées à un trigger, il est impératif de commenter le développement effectué à l'intérieur du code SQL afin de comprendre les différentes étapes ou particularités.**

### Les schémas

  * **Généralité** :
  
Un schéma doit contenir uniquement de la donnée brute qui peut être modifiée soit manuellement ou avec l'aide de déclencheur (ou trigger) mis en place pour automatiser certaines tâches.
Seules les vues pour la gestion ou de filtrage simplifié de la donnée peuvent être contenues dans les schémas de gestion. Les autres vues ayant des usages décisionnels, d'analyses ou d'OpenData sont stockées dans le schémas d'exploitation.

   * **Tableaux de nomage** :

4 types de préfixes de dénomination de schémas sont présents dans la base de données :

|Préfixe|Nom du schéma|Type|Exemple|Définition|
|:-:|:-:|:-:|:-:|:-:|
|met_|nom de la thématique|gestion|met_cul, met_agr, ...|contient des données métiers gérés par la Région Nouvelle-Aquitaine ou utilisées pour les besoins d'un service|
|ref_|nom du référentiel|gestion|ref_ign, ref_gisco, ref_mnhn, ...|contient des données issues de référentiel ou étant concédéré comme des référenties gérées par la Région Nouvelle-Aquitaine de producteurs tiers |
|pro_|nom de la base de données|gestion|pro_sirene, pro_banatic, pro_onisep, ...|contient des données attributaires de référence provenant de producteurs tiers|
|exp_|nom de l'usage|exploitation||contient des données pré-traitées pour les applications WebSIG métiers, Grands Publics, pour des exports OpenData ou des traitements particuliers liés à des projets|
||||exp_apps|schéma contenant des tables ou vues pré-traitées et utilisées dans les applicatifs WebSIG métiers|
||||exp_apps_public|schéma contenant des tables ou vues pré-traitées et utilisées dans les applicatifs Grands Publics|
||||exp_opendata|schéma contenant des tables ou vues pré-traitées et utilisées pour les exports OpenData|
||||exp_projet|schéma contenant des tables ou vues pré-traitées pour répondre à une demande dans le cadre d'un projet|

### Les objets d'un schéma

  * **Généralité** :
  
La dénomination des tables, vues, trigger, function, séquence, index, clé primaire et étrangère ... doit être cohérente entre tous les schémas afin d'assurer une meilleure visibilité des données.
Néanmoins, on peut considérer 2 cas :

. les données de référence : elles sont issues de producteurs extérieurs (comme l'IGN, l'Insee, ...) et dans ces cas particuliers, le nom des tables est conservé afin d'assurer un meilleur suivi,

. les données "dites" métiers sont gérées (pour la plupart) en interne (mais peuvent être d'origine extérieur) et ne sont donc pas soumises à des contraintes de modèle externe. Dans le cas de l'existence d'un format d'échange standard de données, le nom des tables est alors généré à l'export des données.

  * **Les tables** :

Le tableau ci-dessous indique les principes de dénomination des tables. 

|Contenus|Pré-préfixe|Préfixe|exemple|Particularité|
|:-:|:-:|:-:|:-:|:-:|
|données attributaires et géométriques||**geo_**|`geo_p_zone_urba`||
|uniquement de la donnée attributaire ||**an_**|`an_doc_urba`||
|uniquement de la donnée attributaire servant de liens ou de correspondance||**lk_**|`lk_voirie_rurbain`||
|liste de valeur||**lt_**|`lt_typedoc`|Cette table doit contenir au minimum 2 attributs obligatoires : code (codification) et valeur (valeur du code).La valeur du code est une liste ordonnée avec 3 valeurs par défaut (00 : information non renseignée, 99 : valeur autre, ZZ : objet non concerné)|
|log ou information de suivi||**log_**|`log_suivi_audit`||
|traitement applicatif grand public|**xappspublic_**|préfixe correspondant|`xappspublic_an_dec_pav_adr_proxi`||
|traitement applicatif pro|**xapps_**|préfixe correspondant|`xapps_an_fisc_geo_taxe_amgt`||
|export OpenData|**xopendata_**|préfixe correspondant|`xopendata_an_v_bal`||

**Rappel :**

Ces préfixes sont suivis de la dénomination classique des tables.

Cette dénomination peut-être liée à un modèle de données issus d'une norme (cas pour les données des pos-plu) ou laissé à la liberté de l'administrateur en respectant une syntaxe de bases :

**ex :** `geo_[theme]_[identification]`. Si on considère la création d'une table localisant les locaux d'activité, on pourrait la dénommer ainsi geo_eco_locaux.

Les tables doivent être commentées afin d'assurer la compréhension de la donnée (au minimum définir en quelques mots le contenu de la table, la source, l'échelle d'emprise, éventuellement une date de validité, de mise à jour, ...).

**Les attributs génériques d'une table** :

Afin d'éviter les problèmes d'export des données, notamment au format shape, il est recommandé de limiter à 10 caractères le nom des attributs.

La dénomination des attributs reste libre mais doit-être explicite et faire l'objet d'un commentaire.

Seuls certains champs doivent respectés une règle de nommage et doivent être présents dans l'ensemble des tables des données gérées par l'Agglomération lorsque cela est nécessaire :

|attribut|type|définition|
|:-:|:-:|:-:|
|code_insee|character varying(5)|code insee de la commune|
|commune|character varying(150)|libellé de la commune|
|op_sai|character varying(80)|Opérateur de la saisie de la donnée|
|observ|character varying(254)|Commentaires divers|
|date_sai|timestamp without time zone|Horodatage correspondant à la date de saisie de la donnée sans intégration du décalage horaire par rapport au méridient d'origine, valeur non null et par défaut : now()|
|date_maj|timestamp without time zone|Horodatage correspondant à la date de mise à jour de la donnée sans intégration du décalage horaire par rapport au méridient d'origine, à gérer par un trigger before pour update|
|geom||attribut contenant la géométrie|
|sup_m2|integer|Superficie en m²|
|sup_ha|real|Superficie en ha|
|long_m|integer|longueur en mètre|

Pour les données ponctuelles devant être communiquées à l'extérieur en intégrant des champs X/Y, les attributs suivants peuvent être ajoutés : 

|attribut|type|définition|
|:-:|:-:|:-:|
|x_l93|numeric(9,2)| coordonnées X en lambert 93 arrondie au cm (ex : 696000.52)|
|y_l93|numeric(10,2)| coordonnées y en lambert 93 arrondie au cm (ex : 6952300.89)|
|x_wgs84|numeric(2,7)| longitude|
|y_wgs84|numeric(1,9)| latitude|


 * **Les vues** :
 
QGIS imposant une contrainte aux vues pour être affichée, à savoir qu'un identifiant doit être présent et de type entier (integer / serial), il faut penser à ajouter si nécessaire un compteur arbitraire au début de la requête SELECT (ROW_NUMBER() OVER())::integer AS gid.

Il est préférable de forcer le type de géométrie dans la vue pour être correctement intégrée dans geometry_column avec ce paramètre collé à l'attribut de géométrie (geom) `::geometry(polygon,2154)` par ex.

Le tableau ci-dessous indique les principes de dénomination des vues qui découlent de celui des tables. 

|cas|Contenus|Pré-préfixe|Préfixe|exemple|
|:-:|:-:|:-:|:-:|:-:|
|1|données attributaires et géométriques||**geo_v_**|`geo_v_docurba`|
|2|uniquement de la donnée attributaire ||**an_v_**|`an_v_docurba_arcba`|
|3|vue matérilaisée de données attributaires et géométriques||**geo_vmr_**|utilisée uniquement avec les cas 7,8 et 9|
|4|vue matérilaisée en table de données attributaires et géométriques||**geo_vm_**|utilisée uniquement avec les cas 7,8 et 9|
|5|vue matérilaisée de données attributaires||**an_vmr_**|utilisée uniquement avec les cas 7,8 et 9 |
|6|vue matérilaisée en table de données attributaires||**an_vm_**|utilisée uniquement avec les cas 7,8 et 9|
|7|traitement applicatif grand public|**xappspublic_** |préfixe correspondant|`xappspublic_an_vmr_fichegeo_ruplu0_gdpublic`|
|8|traitement applicatif pro|**xapps_** |préfixe correspondant|`xapps_an_v_troncon`|
|9|export OpenData|**xopendata_** |préfixe correspondant|`xopendata_an_v_bal`|

**Rappel :**

Ces préfixes sont suivis de la dénomination classique des tables.

Par défaut, les tables de liens ou de listes de valeur ne peuvent pas faire l'objet d'une vue.

La dénomination des vues doit intégrer l'aspect "emprise géographique" concernée par cette vue. Ex : `geo_v_zone_urba_compiegne` => vue géographique des zonages PLU sur la commune de Compiègne

ATTENTION : Les vues peuvent être commentées, mais l'action de relancer le code CREATE OR REPLACE VIEW sans intégrer la commande COMMENT ON VIEW supprimera le commentaire déjà intégré.

 
 * **Autres objets** :
 
 Le tableau ci-dessous indique les principes de dénomination des autres objets. 

|Objets|Préfixe|Libellé|suffixe|exemple|Particularité|
|:-:|:-:|:-:|:-:|:-:|:-:|
|index||[nom_table]_[champ indexé]|_idx|`geo_p_zone_urba_geom_idx`||
|séquence||[nom_table]_[champ séquence]|_seq|`geo_a_zone_urba_gid_seq`||
|clé primaire||[nom_table]|_pkey|`geo_p_zone_urba_pkey`||
|clé étrangère||[nom_table]_[champ clé(si nécessaire)]|_fkey|`lt_destdomi_fkey`||
|function trigger (générique)|ft_r_|[nom]||`ft_r_l_surf_cal_ha()`|ils sont placés dans le schéma `public`|
|function trigger (spécifique à une table ou vue)|ft_m_|[nom table]|_[type d'éxécution]|`ft_m_an_doc_urba_null()`|ils sont placés dans le schéma principal d'activation|
|trigger|t_t(+n° d'ordre d'éxécution)_|[nom fonction générique] ou [nom_table]_[attribut concerné ou action]||`t_t1_ft_r_l_surf_cal` ou `t_t1_an_doc_urba_null`||

