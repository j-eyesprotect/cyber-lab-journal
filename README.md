# Cyber-Lab Journal

Journal de bord technique de mon labo de cybersecurite/reseau (**CYBER-LAB**) et de ma reconversion vers un poste de **Technicien Systemes et Reseaux (TSSR)**, menee en parallele d'une formation a la Wild Code School.

## Objectif

Documenter, au fil de l'eau, les choix d'architecture, les manipulations techniques, les pannes rencontrees et leur resolution :
- pour garder une trace exploitable de ce que j'apprends,
- - pour presenter un parcours concret et verifiable a de futurs employeurs/formateurs.
 
  - Chaque dossier contient un ou plusieurs documents au format : contexte, mise en place, problemes rencontres, resolution, ce que ca m'a appris.
 
  - ## Stack / environnement du labo
 
  - - Hyperviseur : VirtualBox
    - - Linux : Ubuntu Server (Ubuntu-DC, Ubuntu-01), reseau interne isole du PC hote
      - - Windows : Windows Server 2022 (Active Directory Domain Services), Windows 11 (poste client)
        - - Annuaire : Active Directory, domaine `cyberlab.local`
          - - Partage de fichiers : Samba, integre a l'annuaire AD (realmd / sssd / winbind)
            - - Logging centralise : rsyslog + logrotate
              - - Simulation reseau : Cisco Packet Tracer
               
                - ## Sommaire
               
                - - [`01-infrastructure/`](./01-infrastructure) — Architecture VirtualBox, reseaux internes, VMs du labo CYBER-LAB
                  - - [`02-administration-linux/`](./02-administration-linux) — Utilisateurs/groupes, permissions, sudoers
                    - - [`03-active-directory/`](./03-active-directory) — AD DS, unites d'organisation, groupes, GPO
                      - - [`04-samba-fileserver/`](./04-samba-fileserver) — Partages de fichiers par departement
                        - - [`05-linux-ad-integration/`](./05-linux-ad-integration) — Integration Linux-AD (realmd/sssd/winbind)
                          - - [`06-logging-centralise/`](./06-logging-centralise) — Centralisation des logs avec rsyslog
                            - - [`07-ssh-hardening/`](./07-ssh-hardening) — Authentification SSH par cle, durcissement
                              - - [`packet-tracer/`](./packet-tracer) — Topologie reseau, modele OSI, routage
                               
                                - ## Contexte
                               
                                - En reconversion professionnelle vers la cybersecurite / les reseaux, candidat admis au bootcamp **TSSR (titre RNCP niveau 5)** de la Wild Code School. En attendant la rentree, je construis et fais evoluer un labo personnel complet pour mettre en pratique les notions theoriques et me confronter a de vraies pannes.
                               
                                - ---
                                *Depot mis a jour au fil des sessions de travail sur le labo.*
                                
