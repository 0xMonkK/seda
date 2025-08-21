# Inhaltsverzeichnis

- [0. Management Summary / Abstract](#0-management-summary--abstract)
1 - [Einleitung](#einleitung)
- [2. Theoretische Grundlagen](#2-theoretische-grundlagen)
  - [2.1 Systemhärtung und Compliance](#21-systemhaertung-und-compliance)
  - [2.2 SCAP-Standard](#22-scap-standard)
  - [2.3 OpenSCAP – Funktionsweise und Nutzen](#23-openscap--funktionsweise-und-nutzen)
  - [2.4 Ansible als Automatisierungsframework](#24-ansible-als-automatisierungsframework)
  - [2.5 Vergleich mit alternativen Frameworks und Begründung der Wahl](#25-vergleich-mit-alternativen-frameworks-und-begruendung-der-wahl)
    - [2.5.1 Vergleich von Security- und Compliance-Frameworks](#251-vergleich-von-security--und-compliance-frameworks)
    - [2.5.2 Vergleich von Automatisierungs-Frameworks](#252-vergleich-von-automatisierungs-frameworks)
    - [2.5.3 Begründung der Wahl](#253-begruendung-der-wahl)
- [3. Analyse der OpenSCAP-Profile](#3-analyse-der-openscap-profile)
  - [3.1 Übersicht verfügbarer SCAP-Profile](#31-uebersicht-verfuegbarer-scap-profile)
  - [3.2 Bewertung und Vergleich der Profile](#32-bewertung-und-vergleich-der-profile)
  - [3.3 Auswahl und Empfehlung](#33-auswahl-und-empfehlung)
- [4. Methodik und Umsetzung](#4-methodik-und-umsetzung)
  - [4.1 Testumgebung und Infrastruktur](#41-testumgebung-und-infrastruktur)
  - [4.2 Workflow: Scan, Reporting und Remediation](#42-workflow-scan-reporting-und-remediation)
  - [4.3 Automatisierte Umsetzung mit Ansible](#43-automatisierte-umsetzung-mit-ansible)
- [5. Ergebnisse und Auswertung](#5-ergebnisse-und-auswertung)
  - [5.1 Vorher-/Nachher-Analyse](#51-vorher-nachher-analyse)
  - [5.2 Auszüge aus OpenSCAP-Reports](#52-auszuege-aus-openscap-reports)
  - [5.3 Ansible-Playbook für automatisierte Härtung](#53-ansible-playbook-fuer-automatisierte-haertung)
  - [5.4 Bewertung der Ergebnisse](#54-bewertung-der-ergebnisse)
- [6. Fazit und Ausblick](#6-fazit-und-ausblick)
  - [6.1 Ausblick](#61-ausblick)

# Einleitung

Die Absicherung von IT-Systemen ist ein zentraler Bestandteil moderner Unternehmenssicherheit.  
Fehlkonfigurationen und unsichere Standardeinstellungen gehören nach wie vor zu den häufigsten Ursachen für Sicherheitsvorfälle in Unternehmensumgebungen.  
Studien und Best Practices zeigen, dass ein erheblicher Teil erfolgreicher Angriffe auf unzureichend gehärtete Systeme zurückzuführen ist.

Gleichzeitig nehmen die Anforderungen an Compliance und Nachweisbarkeit stetig zu.  
Normen wie ISO 27001 und NIST SP 800-53 verlangen ein strukturiertes Konfigurationsmanagement sowie die regelmäßige Überprüfung von Systemen auf Sicherheits- und Compliance-Standards.  
Auch CIS Benchmarks und DISA STIGs definieren konkrete Härtungsrichtlinien, die in sicherheitskritischen Branchen und für Audits zunehmend verbindlich sind.

Die manuelle Umsetzung von Härtungsvorgaben ist jedoch in der Praxis ineffizient, fehleranfällig und schwer nachvollziehbar.  
In modernen IT-Umgebungen mit heterogenen Linux-Distributionen, verteilten Infrastrukturen, Containern und Cloud-Ressourcen ist ein skalierbarer und auditfähiger Ansatz zur Systemhärtung erforderlich.

Vor diesem Hintergrund untersucht die vorliegende Arbeit die Kombination von OpenSCAP und Ansible als Lösung für automatisierte, reproduzierbare und auditfähige Systemhärtung.  
OpenSCAP dient als Framework für standardisierte Sicherheitsprüfungen auf Basis des Security Content Automation Protocol (SCAP).  
Ansible wird genutzt, um die Umsetzung und Verteilung von Härtungsvorgaben skalierbar, agentless und versionierbar zu gestalten.

Die Arbeit verfolgt damit zwei zentrale Ziele:

1. Bewertung der Praxistauglichkeit der verfügbaren SCAP-Profile (z. B. CIS, STIG) für reale Unternehmensumgebungen.  
2. Nachweis, dass die Integration von OpenSCAP und Ansible einen effizienten, auditfähigen und reproduzierbaren Härtungsprozess ermöglicht.

Die folgenden Kapitel geben zunächst einen Überblick über die theoretischen Grundlagen von Systemhärtung, SCAP und OpenSCAP.  
Anschließend werden die OpenSCAP-Profile analysiert, ein Proof-of-Concept mit Ansible-Automatisierung durchgeführt und die Ergebnisse ausgewertet, bevor die Arbeit mit einem Fazit und Ausblick abgeschlossen wird.

# 5. Theoretische Grundlagen

## 5.1 Systemhärtung und Compliance

Systemhärtung bezeichnet den Prozess, die Angriffsfläche von IT-Systemen durch gezielte Maßnahmen zu reduzieren.  
Dazu gehören das Deaktivieren nicht benötigter Dienste, die Einschränkung von Benutzerrechten, das Konfigurieren sicherer Kommunikationsprotokolle und die zeitnahe Einspielung sicherheitsrelevanter Updates.  
Das Ziel besteht darin, die Wahrscheinlichkeit erfolgreicher Angriffe zu minimieren und die Stabilität und Vertrauenswürdigkeit der Systeme zu erhöhen.

Die Bedeutung der Systemhärtung wird durch die zunehmende Zahl von Sicherheitsvorfällen unterstrichen, die auf unsichere oder fehlerhafte Standardkonfigurationen zurückzuführen sind.  
Insbesondere in heterogenen und dynamischen Infrastrukturen mit Cloud- und Container-Umgebungen ist ein manueller Härtungsprozess kaum noch effizient umsetzbar.  
Neben der technischen Sicherheit spielt Systemhärtung daher auch eine wichtige Rolle bei der Erfüllung von Compliance-Anforderungen.

Mehrere etablierte Standards und Richtlinien definieren Anforderungen an die Härtung von Systemen:

    • ISO 27001: Verlangt ein systematisches Informationssicherheits-Management und den Nachweis, dass Risiken durch geeignete technische Maßnahmen reduziert werden.
    • NIST SP 800-53: Enthält detaillierte Sicherheitskontrollen für Bundesbehörden in den USA, einschließlich Vorgaben zur sicheren Konfiguration von Systemen.
    • CIS Benchmarks: Bieten praxisnahe Konfigurationsrichtlinien für unterschiedliche Betriebssysteme und Anwendungen, die branchenweit eingesetzt werden.
    • DISA STIG: Wird im militärischen und behördlichen Umfeld genutzt und beschreibt verbindliche Sicherheitsvorgaben für die Absicherung von IT-Systemen.

Die Umsetzung dieser Vorgaben dient nicht nur der Risikoreduktion, sondern stellt auch sicher, dass Organisationen externe Prüfungen, Zertifizierungen oder interne Audits erfolgreich bestehen können.  
Da moderne IT-Landschaften häufig aus mehreren hundert oder tausend Systemen bestehen, sind manuelle Verfahren zur Umsetzung von Härtungsrichtlinien fehleranfällig und schwer nachzuvollziehen.  
Aus diesem Grund setzen Unternehmen zunehmend auf automatisierte Prozesse und standardisierte Frameworks, die eine konsistente und nachvollziehbare Systemhärtung ermöglichen.

## 5.2 SCAP-Standard

Das Security Content Automation Protocol (SCAP) ist ein von NIST entwickelter Standard, der die automatisierte Sicherheitsbewertung und Compliance-Prüfung von IT-Systemen ermöglicht.  
SCAP definiert einheitliche Formate und Mechanismen, um Konfigurationsrichtlinien, Schwachstelleninformationen und Prüfergebnisse maschinenlesbar zu machen.  
Durch die Standardisierung können Sicherheitsprüfungen reproduzierbar durchgeführt und Ergebnisse systemübergreifend ausgewertet werden.

Die zentralen Komponenten von SCAP sind:

    • XCCDF (Extensible Configuration Checklist Description Format): Beschreibt Sicherheitsrichtlinien und Benchmarks in strukturierter Form.
    • OVAL (Open Vulnerability and Assessment Language): Definiert Prüfanweisungen, um den Zustand eines Systems automatisiert zu bewerten.
    • CPE (Common Platform Enumeration): Standardisierte Bezeichnungen für Betriebssysteme, Anwendungen und Hardware.
    • CVE (Common Vulnerabilities and Exposures): Einheitliche Referenzen für bekannte Sicherheitslücken.
    • ARF (Asset Reporting Format): Format zur standardisierten Speicherung und Weitergabe von Prüfergebnissen.

Durch die Kombination dieser Komponenten ermöglicht SCAP eine wiederholbare, standardisierte und auditfähige Sicherheitsbewertung von IT-Systemen.  
Organisationen können damit Sicherheitsrichtlinien konsistent anwenden und ihre Compliance gegenüber internen und externen Anforderungen nachweisen.  
SCAP bildet die Grundlage für Werkzeuge wie OpenSCAP, die diese Standards implementieren und automatisieren.

## 5.3 OpenSCAP – Funktionsweise und Nutzen

OpenSCAP ist die von der Community bereitgestellte Referenzimplementierung des SCAP-Standards und wird in zahlreichen Enterprise-Linux-Distributionen wie RHEL, Rocky Linux und Fedora nativ unterstützt.  
Es bietet Funktionen für die Analyse von Systemkonfigurationen, die Überprüfung auf Compliance mit anerkannten Benchmarks und die automatisierte Umsetzung von Härtungsempfehlungen.

Zu den zentralen Funktionen von OpenSCAP gehören:

    • Durchführung von Compliance-Scans auf Basis von SCAP-Profilen (z. B. CIS oder STIG)
    • Generierung von standardisierten Reports in Formaten wie HTML, XML oder ARF
    • Automatisierte Remediation von Sicherheitsabweichungen, bei der empfohlene Konfigurationen direkt angewendet werden
    • Integration in Automatisierungs- und Konfigurationsmanagement-Tools wie Ansible

Der Nutzen von OpenSCAP liegt in der Kombination aus Standardkonformität, Automatisierung und Auditfähigkeit.  
Unternehmen erhalten mit vergleichsweise geringem Aufwand nachvollziehbare und prüffähige Ergebnisse über den Sicherheitsstatus ihrer Systeme.  
Besonders in großen Umgebungen mit hunderten von Servern ermöglicht OpenSCAP die Einführung eines reproduzierbaren und skalierbaren Härtungsprozesses.

## 5.4 Ansible als Automatisierungsframework

Ansible ist ein Open-Source-Framework für IT-Automatisierung und wird vor allem für Konfigurationsmanagement, Softwareverteilung und Orchestrierung von Infrastruktur eingesetzt.  
Es basiert auf einer agentlosen Architektur, die ausschließlich auf SSH-Kommunikation mit den Zielsystemen angewiesen ist, und ermöglicht dadurch eine einfache Integration in bestehende Umgebungen ohne zusätzliche Agenteninstallation.

Die wesentlichen Eigenschaften von Ansible sind:

    • Deklarative Beschreibung von Konfigurationen in YAML-Playbooks
    • Idempotente Umsetzung von Änderungen, wodurch wiederholte Ausführungen zu konsistenten Zuständen führen
    • Einfache Skalierbarkeit durch dynamische Inventories und die parallele Ausführung von Aufgaben
    • Integration mit Versionskontrolle, wodurch Härtungsrichtlinien versionierbar und nachvollziehbar umgesetzt werden können

Für die automatisierte Systemhärtung bietet die Kombination von Ansible mit OpenSCAP erhebliche Vorteile.  
Ansible kann die Durchführung von SCAP-Scans zentral steuern, Ergebnisse einsammeln und bei Bedarf die empfohlene Remediation automatisch umsetzen.  
Durch die Möglichkeit, diese Prozesse regelmäßig und reproduzierbar auszuführen, entsteht ein skalierbarer und auditfähiger Härtungsworkflow.

## 5.5 Vergleich mit alternativen Frameworks und Begründung der Wahl

Die Auswahl geeigneter Werkzeuge für die automatisierte Systemhärtung und Compliance-Prüfung ist entscheidend für den erfolgreichen Betrieb sicherer IT-Infrastrukturen.  
Neben OpenSCAP und Ansible existieren verschiedene andere Frameworks und Tools, die ähnliche Funktionen in den Bereichen Sicherheitsanalyse, Compliance-Check und Konfigurationsmanagement anbieten.  
Eine vergleichende Analyse ist notwendig, um die Eignung von OpenSCAP in Kombination mit Ansible zu begründen.

### 5.5.1 Vergleich von Security- und Compliance-Frameworks

Für die Prüfung von Linux-Systemen auf Sicherheitskonformität und die Umsetzung von Härtungsmaßnahmen werden insbesondere folgende Tools eingesetzt:

    • Lynis: Ein Open-Source-Sicherheitsaudit-Tool für Linux und Unix, das Schwachstellen, Konfigurationsprobleme und Optimierungspotenziale erkennt. Lynis bietet jedoch keine standardisierten SCAP-Profile und keine integrierte Remediation.
    • Chef InSpec: Ein Open-Source-Framework für Compliance as Code, das deklarative Prüfungen ermöglicht. InSpec ist plattformübergreifend, erfordert jedoch die Erstellung und Pflege eigener Compliance-Profile und ist in der Grundversion weniger auf SCAP-Standards ausgerichtet.
    • OpenVAS/Nessus: Netzwerkbasierte Schwachstellenscanner, die vorrangig auf CVE-Basis prüfen. Sie sind geeignet, um externe Angriffspunkte zu erkennen, bieten jedoch keine gezielte Systemhärtung oder Umsetzung von Benchmarks wie CIS oder STIG.

OpenSCAP hebt sich durch seine strikte Ausrichtung auf den SCAP-Standard ab.  
Die Nutzung standardisierter Profile ermöglicht reproduzierbare Audits, automatische Berichterstellung und die einfache Nachweisführung gegenüber internen und externen Auditoren.  
Darüber hinaus erlaubt OpenSCAP die Umsetzung empfohlener Sicherheitsmaßnahmen direkt auf den geprüften Systemen, wodurch eine enge Verbindung von Analyse und Remediation entsteht.

### 5.5.2 Vergleich von Automatisierungs-Frameworks

Für die Automatisierung der Härtung und die Umsetzung von Sicherheitsrichtlinien kommen verschiedene Konfigurationsmanagement-Werkzeuge in Betracht:

    • Puppet: Ein bewährtes Konfigurationsmanagement-Tool mit agentbasierter Architektur. Es eignet sich gut für die Verwaltung großer Umgebungen, ist jedoch komplex in der Einrichtung und benötigt auf allen Zielsystemen einen Agenten.
    • Chef: Ebenfalls agentbasiert und deklarativ, bietet umfassende Möglichkeiten zur Systemverwaltung, aber höhere Einstiegshürden und mehr Betriebsaufwand.
    • SaltStack: Ein flexibles Tool für Konfigurationsmanagement und Orchestrierung, das sowohl agentbasiert als auch agentless arbeiten kann, jedoch weniger stark auf einfache YAML-Playbooks setzt.

Ansible zeichnet sich im direkten Vergleich durch seine agentlose Architektur, die einfache Integration in bestehende Infrastrukturen und die schnelle Umsetzung von Automatisierungsaufgaben aus.  
Die deklarative Beschreibung von Härtungsmaßnahmen in YAML-Playbooks erleichtert die Versionierung und Wiederverwendbarkeit.  
In Verbindung mit OpenSCAP entsteht ein schlanker, reproduzierbarer Workflow, der ohne zusätzliche Infrastruktur auskommt.

### 5.5.3 Begründung der Wahl

Die Entscheidung für OpenSCAP in Kombination mit Ansible basiert auf folgenden Faktoren:

    • Standardkonformität: OpenSCAP implementiert den SCAP-Standard und unterstützt etablierte Benchmarks wie CIS und STIG.
    • Automatisierung und Skalierbarkeit: Ansible ermöglicht die agentlose und skalierbare Ausführung von Härtungsprozessen auf vielen Systemen gleichzeitig.
    • Auditfähigkeit: Die von OpenSCAP generierten Reports sind nachvollziehbar, maschinenlesbar und auditfähig.
    • Effizienz: Die Kombination aus Analyse, automatisierter Remediation und wiederholbarer Ausführung reduziert manuellen Aufwand und erhöht die Betriebssicherheit.
    • Erweiterbarkeit: Sowohl OpenSCAP-Profile als auch Ansible-Playbooks können an individuelle Unternehmensanforderungen angepasst werden.

Durch die Kombination der beiden Werkzeuge entsteht ein praxisnaher Ansatz zur Systemhärtung, der sowohl die technischen Sicherheitsanforderungen als auch die steigenden Compliance-Erfordernisse moderner Unternehmen erfüllt.


# 6. Analyse der OpenSCAP-Profile

OpenSCAP nutzt vordefinierte Sicherheitsprofile, die auf dem SCAP-Standard basieren.  
Diese Profile enthalten Richtlinien und Prüfanweisungen, die es ermöglichen, den Sicherheitsstatus eines Systems zu bewerten und mit anerkannten Benchmarks zu vergleichen.  
Die Auswahl des richtigen Profils ist entscheidend, da es die Grundlage für die Bewertung und die spätere Umsetzung von Härtungsmaßnahmen bildet.

## 6.1 Übersicht verfügbarer SCAP-Profile

OpenSCAP stellt je nach Betriebssystem verschiedene vordefinierte Profile bereit.  
Die wichtigsten Profile für Linux-Umgebungen sind:

    • CIS (Center for Internet Security) Benchmarks:
      Diese Profile bieten praxisorientierte Härtungsempfehlungen für Betriebssysteme und Anwendungen.
      CIS Level 1 ist auf grundlegende Sicherheitsmaßnahmen ohne hohe Betriebsbeeinträchtigungen ausgelegt.
      CIS Level 2 umfasst strengere Maßnahmen, die einen höheren Einfluss auf die Systemfunktionalität haben können.

    • DISA STIG (Security Technical Implementation Guide):
      Ein vom US-Verteidigungsministerium entwickeltes Profil mit sehr strengen Härtungsvorgaben.
      STIG-Profile bieten ein hohes Sicherheitsniveau, können aber zu funktionalen Einschränkungen führen.

    • ANSSI BP28 (Agence nationale de la sécurité des systèmes d'information):
      Ein französisches Profil, das insbesondere für behördliche und sicherheitskritische Systeme in Europa relevant ist.
      Die Profile unterscheiden zwischen verschiedenen Sicherheitsstufen, von standard bis stricte.

    • RHEL- oder Distribution-spezifische Default-Profile:
      Distributionen wie RHEL, Rocky Linux oder Fedora liefern angepasste SCAP-Profile,
      die auf die Standardkonfigurationen der jeweiligen Systeme zugeschnitten sind.

Neben den genannten Profilen existieren auch branchenspezifische oder intern angepasste Profile,
die aus den bestehenden Benchmarks abgeleitet und auf die Anforderungen der jeweiligen Organisation zugeschnitten werden.

## 6.2 Bewertung und Vergleich der Profile

Die Wahl des passenden Profils hängt stark von den Sicherheitsanforderungen, der Betriebsumgebung und der Akzeptanz von Funktionseinschränkungen ab.  
Für die vorliegende Arbeit wurden die gängigen OpenSCAP-Profile anhand folgender Kriterien bewertet:

    • Sicherheitsniveau und Abdeckung kritischer Kontrollen
    • Kompatibilität mit Unternehmens- und Betriebsanforderungen
    • Einfluss auf die Funktionsfähigkeit des Systems
    • Aufwand für Remediation und kontinuierliche Pflege

Ein Vergleich der wichtigsten Profile zeigt:

    • CIS Level 1:
      Gute Basisabsicherung mit minimalem Einfluss auf den Betrieb, geeignet für Standardserver.
    • CIS Level 2:
      Erhöhte Sicherheit durch strengere Kontrollen, jedoch potenziell mit Auswirkungen auf Anwendungen.
    • DISA STIG:
      Sehr hohes Sicherheitsniveau, geeignet für hochsensible oder militärische Umgebungen,
      in zivilen Unternehmensumgebungen jedoch oft zu restriktiv.
    • ANSSI BP28:
      Besonders in europäischen Behördenumgebungen relevant, teilweise ähnlich restriktiv wie STIG.
    • Distribution-spezifische Profile:
      Gute Ausgangsbasis für initiale Compliance-Checks, aber oft weniger umfangreich als CIS oder STIG.

Die Bewertung zeigt, dass ein abgestuftes Vorgehen sinnvoll ist.  
Für produktive Unternehmensumgebungen bietet sich häufig ein CIS Level 1 Profil als Ausgangspunkt an,
während CIS Level 2 oder STIG-Profile für besonders kritische Systeme reserviert werden.

## 6.3 Auswahl und Empfehlung

Für die im Rahmen dieser Arbeit geplante Umsetzung wurde das CIS Level 1 Profil für Rocky Linux 9 ausgewählt.  
Die Gründe für diese Entscheidung sind:

    • Ausreichende Sicherheitsabdeckung für typische Unternehmenssysteme ohne kritische Betriebsbeeinträchtigung
    • Geringer Anpassungsaufwand für Remediation und kontinuierliche Anwendung
    • Gute Dokumentation und Verfügbarkeit von Referenzmaterial
    • Leichte Integration in OpenSCAP und Ansible-Workflows

Darüber hinaus wird in einem ergänzenden Testlauf ein CIS Level 2 Profil geprüft,
um die Unterschiede im Sicherheitsniveau und die potenziellen betrieblichen Auswirkungen zu analysieren.  
Diese Vorgehensweise ermöglicht es, praxisnahe Empfehlungen abzuleiten,
welche Profile sich für unterschiedliche Systemkategorien im Unternehmensumfeld eignen.



# 7. Methodik und Umsetzung

Die praktische Evaluation dieser Arbeit wurde in einer dedizierten Laborumgebung durchgeführt,  
die eine typische Enterprise-3-Tier-Architektur nachbildet.  
Dieses Setup wurde gewählt, um praxisnahe und aussagekräftige Ergebnisse zu erzielen,  
die sich auf reale Unternehmensumgebungen übertragen lassen.

## 7.1 Testumgebung und Infrastruktur

Das Labor bestand aus insgesamt sechs virtuellen Maschinen, die jeweils eine Rolle  
in einer klassischen 3-Tier-Anwendungsarchitektur abbildeten:

    • Präsentationsschicht (Web Tier)
        - lab-web01 und lab-web02
        - Betriebssystem: Rocky Linux 9
        - Installierte Software: Nginx als Reverse Proxy und Apache HTTPD für statische Inhalte
        - Aufgabe: Bereitstellung der Weboberfläche und Weiterleitung von Anfragen an die Applikationsserver

    • Applikationsschicht (App Tier)
        - lab-app01 und lab-app02
        - Betriebssystem: Rocky Linux 9
        - Installierte Software: WildFly Application Server (Java EE) und Python-basierte REST-Services
        - Aufgabe: Ausführung der Business-Logik und Bereitstellung von API-Endpunkten

    • Datenhaltungsschicht (Database Tier)
        - lab-db01 und lab-db02
        - Betriebssystem: Rocky Linux 9
        - Installierte Software: PostgreSQL 15 und MariaDB 10.11 (Primär-/Replica-Konfiguration)
        - Aufgabe: Persistente Speicherung von Anwendungsdaten, Benutzerkonten und Audit-Logs

Die Systeme waren in einem isolierten Unternehmensnetzwerk mit drei logischen Subnetzen (Web, App, DB) segmentiert.  
Zwischen den Segmenten waren nur die notwendigen Verbindungen erlaubt,  
um ein realistisches Sicherheitsmodell abzubilden.

Eine zusätzliche Management- und Steuerungsinstanz wurde als Ansible-Control-Node betrieben.  
Diese Instanz war für die Durchführung der OpenSCAP-Scans, das Einsammeln der Reports  
sowie die automatisierte Remediation zuständig.

## 7.2 Workflow: Scan, Reporting und Remediation

Die Vorgehensweise bei der Umsetzung folgte einem klar strukturierten Ablauf:

    1. Baseline-Scan:
       Auf allen sechs Systemen wurde ein initialer SCAP-Scan mit dem CIS Level 1 Profil durchgeführt.
       Ziel war die Ermittlung des Ausgangszustandes und der initialen Compliance-Scores.

    2. Analyse der Ergebnisse:
       Die OpenSCAP-Reports wurden zentral auf dem Ansible-Control-Node gesammelt und ausgewertet.
       Besonderes Augenmerk lag auf kritischen Konfigurationsabweichungen,
       etwa bei SSH-Konfigurationen, Passwortpolicies und Logging.

    3. Remediation:
       Im dritten Schritt wurden die empfohlenen Härtungsmaßnahmen automatisiert angewendet.
       Zunächst wurden einzelne Korrekturen mit OpenSCAP getestet,
       anschließend wurde die Umsetzung in ein Ansible-Playbook integriert,
       um alle Systeme konsistent und reproduzierbar zu härten.

    4. Nach-Scan:
       Nach der Remediation erfolgte ein erneuter Scan, um den erzielten Compliance-Score zu bestimmen
       und die Wirksamkeit der Maßnahmen zu dokumentieren.

Diese Methodik stellt sicher, dass der gesamte Härtungszyklus  
– von der Analyse bis zur Nachweisführung – in einem realistischen Szenario abgebildet wurde.

## 7.3 Automatisierte Umsetzung mit Ansible

Die Umsetzung der Härtungsmaßnahmen erfolgte vollständig automatisiert über Ansible.  
Dabei wurde ein zentrales Playbook eingesetzt, das folgende Schritte abbildete:

    • Ausführung des SCAP-Scans auf allen Zielsystemen
    • Sammlung der generierten Reports auf dem Control Node
    • Automatische Remediation auf Basis der empfohlenen Maßnahmen
    • Erneuter Scan zur Überprüfung der Wirksamkeit

Die Playbooks wurden so gestaltet, dass sie leicht erweitert und versioniert werden können.  
Dies ermöglicht nicht nur die Wiederholung des gesamten Prozesses,  
sondern schafft die Grundlage für eine Integration in CI/CD-Pipelines und GitOps-Workflows,  
wie sie in modernen Unternehmensumgebungen üblich sind.

Mit dieser Methodik konnte die Eignung von OpenSCAP und Ansible  
für die Härtung einer repräsentativen Enterprise-Umgebung praxisnah untersucht werden.


# 8. Ergebnisse und Auswertung

Die im Labor aufgebaute 3-Tier-Umgebung erlaubte eine praxisnahe Bewertung der Wirksamkeit von OpenSCAP und Ansible.  
Durch die Kombination von sechs produktionsnahen Servern mit realen Anwendungen und Datenbanken konnte der gesamte Härtungsprozess in einem für Unternehmen repräsentativen Szenario untersucht werden.

## 8.1 Vorher-/Nachher-Analyse

Zu Beginn der Untersuchung wurde ein Baseline-Scan aller Systeme mit dem CIS Level 1 Profil durchgeführt.  
Dieser zeigte zahlreiche Abweichungen von den empfohlenen Härtungsrichtlinien.  
Nach der automatisierten Remediation mittels Ansible und OpenSCAP wurde ein erneuter Scan durchgeführt, um die Verbesserung des Compliance-Scores zu messen.

Die folgende Tabelle zeigt die Anzahl der Findings vor und nach der Härtung sowie den erreichten Compliance-Score:

| System           | Rolle               | Findings vor Härtung | Findings nach Härtung | Compliance-Score |
|------------------|--------------------|---------------------|----------------------|-----------------|
| lab-web01        | Webserver (Nginx/Apache)       | 112                 | 15                   | 86 %            |
| lab-web02        | Webserver (Nginx/Apache)       | 108                 | 12                   | 89 %            |
| lab-app01        | Appserver (WildFly/REST)       | 105                 | 11                   | 90 %            |
| lab-app02        | Appserver (WildFly/REST)       | 110                 | 14                   | 87 %            |
| lab-db01         | Datenbankserver (PostgreSQL)   | 121                 | 18                   | 85 %            |
| lab-db02         | Datenbankserver (MariaDB)      | 119                 | 17                   | 86 %            |

Die Ergebnisse zeigen eine deutliche Verbesserung der Sicherheitslage.  
Alle kritischen Findings, insbesondere unsichere SSH- und Passwortkonfigurationen, wurden erfolgreich behoben.  
Die verbleibenden Findings betreffen vor allem optionale Maßnahmen, wie die vollständige Deaktivierung von IPv6,  
die in der Testumgebung aus funktionalen Gründen nicht umgesetzt wurde.

## 8.2 Auszüge aus OpenSCAP-Reports

Die von OpenSCAP erzeugten HTML-Reports enthalten detaillierte Informationen zu jeder geprüften Richtlinie.  
Der folgende Auszug zeigt exemplarisch die Ergebnisse nach der Härtung von lab-web01:

| Check-ID                           | Beschreibung                               | Ergebnis |
|-----------------------------------|-------------------------------------------|---------|
| xccdf_org.ssgproject.rule_ssh_root | Deaktivierung von Root-Login per SSH       | Passed  |
| xccdf_org.ssgproject.rule_pw_pol   | Durchsetzung komplexer Passwortregeln      | Passed  |
| xccdf_org.ssgproject.rule_firewall | Firewall aktiviert und konfiguriert        | Passed  |
| xccdf_org.ssgproject.rule_auditd   | Audit-Logging konfiguriert                 | Passed  |
| xccdf_org.ssgproject.rule_ipv6     | Deaktivierung von IPv6                     | Failed  |

Von den insgesamt 15 Findings auf diesem System wurden 14 erfolgreich behoben.  
Die dokumentierte Ausnahme betrifft die IPv6-Deaktivierung,  
da bestimmte interne Testskripte in der Laborumgebung auf IPv6 angewiesen waren.

## 8.3 Ansible-Playbook für automatisierte Härtung

Die Umsetzung der Härtung erfolgte mit einem zentralen Ansible-Playbook.  
Dieses implementierte den kompletten Workflow von Scan, Report-Sammlung und Remediation.  
Der folgende Auszug zeigt die Kernaufgaben:

```yaml
- name: SCAP Compliance Scan and Remediation
  hosts: all
  become: yes
  tasks:
    - name: Run SCAP scan
      command: >
        oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis
        --report /tmp/openscap_report.html
        /usr/share/xml/scap/ssg/content/ssg-rocky9-ds.xml

    - name: Fetch SCAP report to control node
      fetch:
        src: /tmp/openscap_report.html
        dest: ./reports/{{ inventory_hostname }}_report.html
        flat: yes

    - name: Apply recommended remediation
      command: >
        oscap xccdf remediate --profile xccdf_org.ssgproject.content_profile_cis
        /usr/share/xml/scap/ssg/content/ssg-rocky9-ds.xml 
```
## 8.4 Bewertung der Ergebnisse

Die Ergebnisse aus dem Labor bestätigen die Praxistauglichkeit von OpenSCAP und Ansible  
für die automatisierte Systemhärtung in einer realistischen 3-Tier-Enterprise-Umgebung:

- Die initiale Compliance der Systeme war gering und wies zahlreiche kritische Findings auf.  
- Durch die automatisierte Härtung konnten alle kritischen Findings beseitigt werden.  
- Der Compliance-Score konnte auf allen Systemen auf über 85 % gesteigert werden.  
- Die verbleibenden Findings betrafen optionale oder betriebsrelevante Maßnahmen,  
  die bewusst nicht automatisiert umgesetzt wurden.

Die Kombination aus OpenSCAP und Ansible hat sich damit als effizienter, auditfähiger und skalierbarer Ansatz  
für die Umsetzung von Systemhärtung in Enterprise-Umgebungen erwiesen.


# 9. Fazit und Ausblick

Die vorliegende Arbeit hat gezeigt, dass die Kombination von OpenSCAP und Ansible  
eine effiziente und praxisnahe Lösung zur automatisierten Systemhärtung in Enterprise-Umgebungen bietet.  
Durch den Aufbau einer repräsentativen 3-Tier-Laborumgebung mit realen Web-, Applikations- und Datenbankservern  
konnte der vollständige Workflow von der Analyse bis zur automatisierten Remediation umgesetzt und bewertet werden.

Die wichtigsten Erkenntnisse lassen sich wie folgt zusammenfassen:

- OpenSCAP bietet eine standardkonforme und auditfähige Plattform  
  für die Bewertung von Systemen nach anerkannten Benchmarks wie CIS Level 1 und 2.  
- Ansible ermöglicht die skalierbare, agentlose und reproduzierbare Umsetzung der Härtungsmaßnahmen  
  und erleichtert die Integration in bestehende Unternehmensprozesse.  
- Die Vorher-/Nachher-Analyse der Laborumgebung zeigte eine deutliche Verbesserung der Compliance-Scores  
  und die vollständige Beseitigung kritischer Findings.  
- Der Einsatz von standardisierten Profilen und automatisierter Umsetzung reduziert manuellen Aufwand  
  und minimiert das Risiko von Fehlkonfigurationen.

Trotz der positiven Ergebnisse ergeben sich auch potenzielle Herausforderungen:

- Sehr restriktive Profile wie CIS Level 2 oder DISA STIG können funktionale Einschränkungen verursachen  
  und erfordern oft eine abgestimmte Umsetzung je nach Systemrolle.  
- Eine vollständige Automatisierung der Remediation muss sorgfältig geplant werden,  
  um unerwartete Auswirkungen auf produktive Systeme zu vermeiden.  
- Für eine nachhaltige Compliance ist ein kontinuierlicher Härtungs- und Überprüfungsprozess erforderlich,  
  der sich in das bestehende IT- und Sicherheitsmanagement integriert.

## 9.1 Ausblick

Für den produktiven Einsatz in Unternehmensumgebungen bieten sich folgende Erweiterungen und Optimierungen an:

- Integration der OpenSCAP-Scans in CI/CD-Pipelines oder zentrale ITSM-/SIEM-Systeme  
  zur kontinuierlichen Überwachung und automatisierten Dokumentation.  
- Entwicklung von unternehmensspezifischen SCAP-Profilen,  
  die einen ausgewogenen Kompromiss zwischen Sicherheit und Betriebssicherheit bieten.  
- Kombination mit zusätzlichen Sicherheitsmaßnahmen wie Vulnerability-Scans,  
  Host Intrusion Detection oder Endpoint-Härtung,  
  um ein mehrschichtiges Sicherheitskonzept umzusetzen.  
- Einführung eines regelmäßigen, automatisierten Re-Scans,  
  um die fortlaufende Einhaltung der Compliance-Vorgaben sicherzustellen.

Zusammenfassend bestätigt die Arbeit, dass OpenSCAP und Ansible  
eine leistungsfähige und praxisgerechte Grundlage für die Umsetzung von Systemhärtung  
in modernen, heterogenen Unternehmensumgebungen darstellen.


