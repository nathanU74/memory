### PHP MD



#### 1. 설치 

- [phpmd.org](https://phpmd.org/download/index.html)



````json
{
    "require-dev": {
        "phpmd/phpmd" : "@stable"
    }
}
````

````bash
composer install
````



![setup phpstorm](https://tva1.sinaimg.cn/large/007S8ZIlgy1geyvz7glrhj30y90q7tfy.jpg)



#### 2. PHPMD 설정 추가

> Avoid variables with short names like $id. Configured minimum length is 3.
>
> 짧은 변수에 대한 오류 피하기 에를 들면 loop를 돌기 위해 주로 사용하는 $i, $j, $k.... 또는 간단한 $id 같은 변수 사용시

composer로 설치 했다면 /vendor/phpmd/src/main/resources/naming.xml 이 존재 한다.

그걸 복사해서 custom-naming.xml 로 변경하고 custom 설정을 저장할 만한 곳에 저장한다.



![naming.xml](https://tva1.sinaimg.cn/large/007S8ZIlgy1geyvk2286yj30dt0ndjst.jpg)



- **xml 코드 수정**

``` xml
<property name="exceptions" description="Comma-separated list of exceptions" value=""/>
```

부분을 주석 처리 하거나 value 값을 수정한다. Id, i, j, k, e

````xml
<?xml version="1.0" encoding="UTF-8"?>
<ruleset name="Naming Rules"
         xmlns="http://pmd.sf.net/ruleset/1.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd"
         xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
    <description>
The Naming Ruleset contains a collection of rules about names - too long, too short, and so forth.
    </description>

    <rule name="ShortVariable"
          since="0.2"
          message="Avoid variables with short names like {0}. Configured minimum length is {1}."
          class="PHPMD\Rule\Naming\ShortVariable"
          externalInfoUrl="https://phpmd.org/rules/naming.html#shortvariable">
        <description>
Detects when a field, local, or parameter has a very short name.
        </description>
        <priority>3</priority>
        <properties>
            <property name="minimum" description="Minimum length for a variable, property or parameter name" value="3"/>
          	<!--<property name="exceptions" description="Comma-separated list of exceptions" value=""/>-->
            <property name="exceptions" description="Comma-separated list of exceptions" value="id,i,j,k,e"/>
        </properties>
        <example>
````



- **PhpStorm에 rule 적용**

![custom-rules](https://tva1.sinaimg.cn/large/007S8ZIlgy1geyvk2dqluj30p609t0ul.jpg)

