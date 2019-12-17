# hubspotApi

## usage : 

#### requirements

include config file with constant :  
 - FORM_BASE_URL (hubspot form api url)  
 - PORTAL_ID  

#### create your own php form class : 
 
 ```php
 namespace eteamsys\hubspot;

use eteamsys\hubspot\form\SubmitAbstract;

class SampleForm extends SubmitAbstract{

    protected $formId = ''; //mandatory

    protected $formName = ''; // send to hubspot

    protected $varsMapping = [
        'titre1'             => ['source' => 'POST' , 'field' => 'titre' , 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
        'nom1'               => ['source' => 'POST' , 'field' => 'lastname' , 'format' =>\eteamsys\hubspot\Converter\UpperCaseConverter::class],
        'prenom1'            => ['source' => 'POST' , 'field' => 'firstname', 'format' =>\eteamsys\hubspot\Converter\UCfirstConverter::class],
        'email1'             => ['source' => 'POST' , 'field' => 'email', 'format' =>\eteamsys\hubspot\Converter\LowerCaseConverter::class],
        'telephone1'         => ['source' => 'POST' , 'field' => 'phone', 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
        'type_support'       => ['source' => 'POST' , 'field' => 'connu_', 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
        'rgpd_consentement1' => ['source' => 'POST' , 'field' => 'rgpd', 'format' =>\eteamsys\hubspot\Converter\BooleanConverter::class],
    ];

}
 ```
 key    : form field name (variable key)  
 source : global variable used (POST GET COOKIE SESSION GLOBAL ENV SERVER Constant availables)  
 field  : hubspot field name  
 format : value transformation (use NullConverter if it isn't needed)  

#### post the form :

 ```php
        /**
	* hubspot form submission
	*/
	include_once('../eteamsys/hubspot/config.php');
	$form = new eteamsys\hubspot\SampleForm();
	$result = $form->setVars()->setPortalId(PORTAL_ID)->submit();
	if(!$result) {
            $error = $form->getError();
	}
  ``` 

## Configure and Customize  : 

### key :

You can put an array subindex.  
For that separate each index by "\".

Exemple :  

```php

#for $_POST['choice'] equals ['car' , 'truck']

//'choice\0'             => ['source' => 'POST' , 'field' => 'choice' , 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],

/** for $HTTP_RAW_POST_DATA equals 
    { 
        "person": { 
            "name": "John Doe", 
            "age": 30, 
            "car": null 
        }
    }
*/

//'person\name'             => ['source' => 'POST' , 'field' => 'fullname' , 'format' =>\eteamsys\hubspot\Converter\UpperCaseConverter::class],

```

### extends your forms : 

you can extends form an other form class. In this case,  varsMapping is merged with parent mapping.  

exemple : 

 ```php

class Sample1Form extends SubmitAbstract{

    protected $formId = ''; //mandatory

    protected $formName = ''; // send to hubspot

    protected $varsMapping = [
        'nom1'               => ['source' => 'POST' , 'field' => 'lastname' , 'format' =>\eteamsys\hubspot\Converter\UpperCaseConverter::class],
        'prenom1'            => ['source' => 'POST' , 'field' => 'firstname', 'format' =>\eteamsys\hubspot\Converter\UCfirstConverter::class],
        'email1'             => ['source' => 'POST' , 'field' => 'email', 'format' =>\eteamsys\hubspot\Converter\LowerCaseConverter::class],
    ];

}

class Sample2Form extends Sample1Form {

    protected $formId = ''; //changed it

    protected $formName = ''; //send to hubspot

    protected $varsMapping = [
        'titre1'             => ['source' => 'POST' , 'field' => 'titre' , 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
        'telephone1'         => ['source' => 'POST' , 'field' => 'phone', 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
    ];

}

$form = new Sample2Form();

var_export($form->getVarsMapping());

/**
* result : 
*
* array(
*    'titre1'             => ['source' => 'POST' , 'field' => 'titre' , 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
*    'telephone1'         => ['source' => 'POST' , 'field' => 'phone', 'format' =>\eteamsys\hubspot\Converter\NullConverter::class],
*    'nom1'               => ['source' => 'POST' , 'field' => 'lastname' , 'format' =>\eteamsys\hubspot\Converter\UpperCaseConverter::class],
*    'prenom1'            => ['source' => 'POST' , 'field' => 'firstname', 'format' =>\eteamsys\hubspot\Converter\UCfirstConverter::class],
*    'email1'             => ['source' => 'POST' , 'field' => 'email', 'format' =>\eteamsys\hubspot\Converter\LowerCaseConverter::class],
* );
*/

 ```


### Availables Source extractor :

 POST      : get value from $_POST  
 GET       : get value from $_GET  
 COOKIE    : get value from $_COOKIE  
 SESSION   : get value from $_SESSION  
 GLOBAL    : get value from $GLOBAL (see https://www.php.net/manual/fr/reserved.variables.globals.php)  
 ENV       : get value from $_ENV simular as getenv()  
 SERVER    : get value from $_SERVER  
 Constant  : get value from global constant   

#### Specials :  

 JSON_HTTP_RAW_POST_DATA : set up an array from $HTTP_RAW_POST_DATA if POST DATA is formated as json (use it for json API's)

#### Custom extractor : 

You can build your own extractor. It mus implements \eteamsys\hubspot\Extractor\ExtractorInterface.  
In this case, put as "source" your extractor full Class name just like formater class

```php

//'titre'             => ['source' => \me\extractor\MyCustomExtracto::class , 'field' => 'titre' , 'format' =>\eteamsys\hubspot\Converter\NullConverter::class

```

Exemple : 

### Availables formater :

 NullConverter           : do nothing  
 BooleanConverter        : return true or false as string  
 FrenchDateConverter     : Concert from french date format DD/MM/YYYY to american format YYYY-MM-DD  
 LowerCaseConverter      : put value as lowercase  
 StringConverter         : to string transtyping  
 UCfirstConverter        : Ucfirst (first character as uppercase)
 UpperCaseConverter      : put value as uppercase  
 ArrayAgregatorConverter : implode array to string separate by ","  
  
