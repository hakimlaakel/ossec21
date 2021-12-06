# ossec21

Auteur : Hakim Laakel

Cours : Administration et sécurité Linux et Windows

Formateur : F-E Goffinet

## SCAP : Security Content Automation Protocol

### *** Preparation ***

• Générer une clé RSA 

• La copier sur la cible (un mot de passe sera demandé une dernière fois) 

• Installer oscap-scanner sur la cible  

 > ssh-keygen -b 4096 -t rsa -f $HOME/.ssh/id_rsa -q -N ""
 
 > target="srv2"
 
 > ssh-copy-id $target
 
 > ssh $target "yum -y install scap-security-guide"


### *** Setup de la station de contrôle ***

> curl -L https://git.io/JMPci \ -o /usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml

Note : Le projet SSG construit des contenus de sécurité dans différents formats : XCCDF, OVAL et Source DataStream.


### *** Scan avec le Data Stream ***

> data_stream="/usr/share/xml/scap/ssg/content/ssg-rhel7-ds-1.2.xml"
 
> profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"
 
> cpe_dict="/usr/share/xml/scap/ssg/content/ssg-rhel7-cpe-dictionary.xml"

> type="$target-bp28m-before"

>  oscap-ssh --sudo root@$target 22 xccdf eval --fetch-remote-resource --profile $profile --results $type-results.xml --report $type-report.html --oval-results --cpe $cpe_dict $data_stream


### *** Vérifier les fichiers générés ***
> ll srv2*


### *** Générer le guide de configuration ***

> oscap xccdf generate guide --profile $profile --output $type-guide.html $type-results.xml


### *** Remédier automatiquement avec Ansible ***

*Exemple : 

*Title*   Ensure Software Patches Installed

*Rule*    xccdf_org.ssgproject.content_rule_security_patches_up_to_date

*Ident*   CCE-26895-3

*Result*  fail

> rule1="xccdf_org.ssgproject.content_rule_security_patches_up_to_date"

> oscap-ssh --sudo root@$target 22 xccdf eval \
 --fetch-remote-resource \
 --profile $profile \
 --results rule1-$type-results.xml \
 --report rule1-$type-report.html \
 --oval-results \
 --cpe $cpe_dict \
 --rule $rule1 \
 
> result_id=$(oscap info rule1-$type-results.xml | grep 'Result ID' | sed 's/[[:blank:]]Result ID: //')
> 
> oscap xccdf generate fix \
--fix-type ansible \
--output rule1-$type-playbook.yml \
--profile $profile \
--result-id $result_id \
rule1-$type-results.xml

ansible-playbook -i "$target," rule1-$type-playbook.yml --check --diff

ansible-playbook -i "$target," rule1-$type-playbook.yml

ansible all -i "$target," -m reboot

### *** Valider la mise en conformité ***

> type="std-after"

> rule1="xccdf_org.ssgproject.content_rule_security_patches_up_to_date"

> profile="xccdf_org.ssgproject.content_profile_anssi_nt28_minimal"

> oscap-ssh --sudo root@$target 22 xccdf eval \
--fetch-remote-resource \
--profile $profile \
--results rule1-$type-results.xml \
--report rule1-$type-report.html \
--oval-results \
--cpe $cpe_dict \
--rule $rule1 \
$data_stream


#### Resultat : 

 Title   Ensure Software Patches Installed

 Rule    xccdf_org.ssgproject.content_rule_security_patches_up_to_date

Ident   CCE-26895-3

Result pass

