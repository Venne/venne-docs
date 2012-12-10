# Testování modulů seleniem

## Konfigurace selenia v config.neon

	parameters:
		selenium:
			browser: *chrome
			browserUrl: http://localhost/sandbox/www
			screenshotPath: %tempDir%/phpunit
			screenshotUrl: http://localhost/sandbox/www

## Soubor `phpunit.xml`

Tento soubor by měl být přímo v rootu modulu. Může vypadat například takto:

    <?xml version="1.0" encoding="utf-8" ?>
    <phpunit
        stopOnError="false"
        stopOnFailure="false"
        stopOnIncomplete="false"
        stopOnSkipped="false"
    
        convertErrorsToExceptions="true"
        convertNoticesToExceptions="true"
        convertWarningsToExceptions="true"
        
        backupGlobals="false"
        backupStaticAttributes="false"
    
        bootstrap="tests/bootstrap.php"
        colors="true"
        >
        <testsuites>
            <testsuite name="CmsModule Test Suite">
                <directory suffix="Test.php">./tests/selenium</directory>
            </testsuite>
        </testsuites>
    </phpunit>

Selenium testy budeme tedy psát na cestě `@myModule/tests/selenium`.

## Spouštění testů

### Start `selenium-server`

#### Archlinux
	java -jar /usr/share/selenium-server/selenium-server-standalone.jar

### Spuštění PhpUnit

- `phpunit` - kompletní test
- `phpunit --filter LanguagePresenterTest` - aplikace filtru

