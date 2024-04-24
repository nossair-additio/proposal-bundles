# Additio Core Proposal Bundles

Tenim un problema anomenat _Additio Core Bundles_ on cada vegada que hem de fer _merge_ de branques o desplegar versions a _dev_ per ser testejades entrem en conflictes constants.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Conflictes](#conflictes)
- [Solució proposada](#soluci%C3%B3-proposada)
  - [Stage 0](#stage-0)
    - [Problemes](#problemes)
- [Demo](#demo)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Conflictes

1. Interrompre el desenvolupament del programador per gestionar aquests bundles.
2. Investigació del equip de QA per veure si està tot pujat i questionar a l'equip de desenvolupament si estàn pujats.
3. El fluxe o el cicle de temps per treure aquestes tasques a producció.
4. Al fer un merge tornar a generar aquests _bundles_ per poder asegurar el seu bon funcionament.

> Entre d'altres més, però aquestes son les més importants segons el meu criteri.

## Solució proposada

**Crear _Releases_ preliminars amb el nom de la branca.**

¿Que significa això?

Actualment el que estem fent es adjuntar els bundles de forma manual, i no fem ús de tècniques de CI/CD per l'entorn de desenvolupament, només per l'entorn de Producció.

El que busco amb aquesta proposta es trancar amb aquesta dependencia i fer-ho tot automatizat menys algun efecte colateral.

### Stage 0

Crear dos workflows nous anomenats `dev-release.yml` i `manual-release.yml`.

> Els noms son perfectament debatibles.

- `dev-release.yml`: Aquest s'executarà un cop hem realitzat un merge a una branca que comenci per `version*` o `modules*`. (Sigui per PR o merge manual).
- `manual-release.yml`: Aquest s'executarà de forma manual, únicament quan necessitem provar una branca a `dev` individual que requereixi de `bundles`.

¿Que fan aquests dos nous workflows?

- `dev-release.yml`: Observarà el nom de la branca `base` on apuntem els nostres canvis i desplegarà una release preliminar o més ben dit una `pre-release` que s'anirà modificant cada vegada que llençem la _Github Action_ en la branca _base_.

- `manual-release.yml`: Fara el mateix que la primera part però de una branca d'un desenvolupador.

> Exemple: Si tenim la branca `1.0.10/CODE-10` farem una `pre-release` anomenada 1.0.10-code-10.

¿Com farem perquè això funcioni en els devs?

Actualment tenim un sistema en els entorns de desenvolupament que cada `10` minuts s'actualitza amb els últims canvis.

La solució seria obtenir el nom de la branca en la que estem i actualitzar el nostre repositori fent un `bower install additio-core@10.x`.

També haurem de fer un `script` per obtenir les imatges corresponents dels `bundles` i afegir-los dintre de les directoris de `frontend`.

Apart de fer-ho quan s'actualitzi també al clicar el botó de desplegament d'aquell entorn de dev.

El nom que obtenim sol ser per exemple un `version/10.x` o `modules/AdditioCoreImprovements-v1.2`, es faria un split entre `/` i s'obtindria la segona part amb la cual es faria el `bower install additio-core`.

> Si tenim alguna branca sense '/' que per cert es una mala pràctica, hauriem d'obtenir el seu codi i fer una release amb ell.

> Per exemple:
> - `bower install additio-core@10.x`
> - `bower install additio-core@additio-core-improvements-v1.2`

¿Com treballarem a local?

1. Eliminarem els bundles de `bower_components`.
2. Ficarem un `gitignore` per evitar detecció de canvis i tornar-los a pujar.
3. Tindrem dos escenaris per actualitzar els bundles:
  - Fer ús del propi sistema actual que tenim els `frontenders` per moure els bundles, aixi podem continuar desenvolupant i fent proves com sempre.
  - Fer ús d'un nou sistema per que els desenvolupadors puguin actualizar els bundles de la seva branca, será un script que indicant el nom de la branca s'actualitzi tot automaticament, tant imatges com bundles de tots llocs. (Pensar una idea una mica més automatizada).

#### Problemes

1. Les _Github Actions_ tenen un cost i ja estem abusant una mica, però crec que ens toca abusar d'elles si volem un esquema CI/CD 'perfecte'.
> La solució aquest problema pot estar en treballar amb un `self-hosting`, un ordinador nostres conectat 24/7 a l'empresa que s'utilitzaria exclusivament (almenys al principi) per les _Github Actions_. (Pot ser un ordinador bastant simple inicialment, d'algun que ja tinguem a l'empresa).
2. Les _Github Actions_ poden fallar i son responsabilitat de cada desenvolupador solucionar-les.
3. Crearem una magnitud bastant alta de `pre-release` que haurem d'anar eliminant un cop hagin sortit.
> Una solució com a Stage 1, seria crear una mena d'eina per poder eliminar les `pre-releases` que formin part de la que ja surt a producció.

## Demo

:smiley:
