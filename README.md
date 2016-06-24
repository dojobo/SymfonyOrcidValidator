# SymfonyOrcidValidator
A Validation Constraint for ORCIDs, for use in Symfony\Validator 3.1

[ORCIDs](http://orcid.org/) are 16-digit unique, persistent identifiers for researchers. They are issued only by ORCID. This validator checks whether an input is a valid ORCID. To be considered a valid ORCID, an input:

- MUST be exactly 16 digits long
- MUST consist of only numbers for the first 15 digits, followed by a final number or the letter `X`
- MUST have the correct checksum as the final digit
- MAY be expressed with hyphens after every four characters, for readability (e.g., `0000-0001-5000-0007`)
- MAY use either case for the letter `X`/`x`
- MAY be expressed as a URI, e.g., `http://orcid.org/0000-0002-1825-0097`

These specifications are fully explained [here](http://support.orcid.org/knowledgebase/articles/116780-structure-of-the-orcid-identifier). For stricter validation, there are also options for case sensitivity and requiring hyphens.

## Requirements
- php 5.6+
- `"symfony/symfony": "~3.1"` or `"symfony/validator": "~3.1"`
- `"phpunit/phpunit": "~5.4"` (for running the tests)

(Other versions might work; I haven't tried them.)

## Installation
Add `symfony/symfony` or `symfony/validator` (in the case of a project using Symfony components, e.g. [Silex](http://silex.sensiolabs.org/)) to your `composer.json`, and run `composer update` and `composer install`.

Currently the two classes are namespaced for my own Symfony/Symfony fork, where I developed them. I recommend putting them in your project's `src` directory (or equivalent, or whatever subdirectory you deem appropriate), and then changing the namespace to match. This should work with [PSR-4 autoloading](http://www.php-fig.org/psr/psr-4/), which I also recommend handling via composer.

## Usage
This Validation Constraint works just like [the others in Symfony/Validation](http://symfony.com/doc/current/book/validation.html#validation-constraints). 

This example assumes that `Orcid` and `OrcidValidator` are in the same namespace as `Person`:

    use Symfony\Component\Validator\Mapping\ClassMetadata;

    class Person
    {
        protected $name;
        protected $nameIdentifier;

        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $metadata->addPropertyConstraint('nameIdentifier', new Orcid());
        }

        public function __construct($name, $nameIdentifier)
        {
            $this->name = $name;
            $this->nameIdentifier = $nameIdentifier;
        }
    }

To validate the class, call `validate($person)` from your app's Validator service. You can look [here](http://symfony.com/doc/current/book/validation.html#using-the-validator-service) for an example in Symfony, or here is an example in Silex:

    $app = new \Silex\Application();
    $app->register(new \Silex\Provider\ValidatorServiceProvider());
    $carl = new Person('Sagan, Carl', '0000-0002-3158-3558');
    $errors = $app['validator']->validate($carl);
    if (count($errors) > 0) {
        return new Response((string) $errors);
    } else {
        return new Reponse('Valid ORCID!');
    }


## Running the tests
Please note that the included tests will not run successfully outside of the Symfony testing harness, unfortunately. If you want to run them yourself:

1. Leave `Orcid.php` and `OrcidValidator.php` with their default namespace of `Symfony\Component\Validator\Constraints`. 
2. Copy both files to `[symfony_src_dir]/Component/Validator/Constraints`.
3. Copy `OrcidValidatorTest.php` to `[symfony_src_dir]/Component/Validator/Tests/Constraints`.
4. Per [these instructions](http://symfony.com/doc/current/contributing/code/tests.html), `cd` to the Symfony root directory and run `phpunit src/Symfony/Component/Validator` to run the whole suite of Validator tests.
