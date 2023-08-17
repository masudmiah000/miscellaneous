# Password Policy Rule for Laravel

This repository contains a custom validation rule class, `PasswordPolicyRule`, that enforces a strong password policy. It also includes PHPUnit unit tests to verify the functionality of the rule.

## Installation

1. Clone this repository to your local machine or download the ZIP archive.

2. Navigate to your Laravel project's root directory using your terminal.

3. Copy the `PasswordPolicyRule.php` file from the `app/Rules` directory of this repository and paste it into your Laravel project's `app/Rules` directory.

```
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class PasswordPolicyRule implements Rule
{
    protected $minLength;

    /**
     * Create a new rule instance.
     *
     * @param  int  $minLength  The minimum required length for the password.
     * @return void
     */
    public function __construct($minLength = 8)
    {
        $this->minLength = $minLength;
    }

    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute  The name of the attribute being validated.
     * @param  mixed  $value  The value of the attribute.
     * @return bool  True if the validation passes, otherwise false.
     */
    public function passes($attribute, $value)
    {
        // Check for minimum length
        if (strlen($value) < $this->minLength) {
            return false;
        }

        // Check for at least one uppercase letter, one lowercase letter, one number, and one special character
        if (!preg_match('/[A-Z]/', $value) || !preg_match('/[a-z]/', $value) || !preg_match('/[0-9]/', $value) || !preg_match('/[!@#\$%^&*()\-_=+\[\]{};:\'",.<>\/?|\\\\]/', $value)) {
            return false;
        }

        // Check for sequential or repeating characters like "123" or "aaa"
        if ($this->hasSequentialOrRepeatingCharacters($value)) {
            return false;
        }

        return true;
    }

    /**
     * Determine if the value contains sequential or repeating characters.
     *
     * @param  string  $value  The value to be checked.
     * @return bool  True if the value contains sequential or repeating characters, otherwise false.
     */
    protected function hasSequentialOrRepeatingCharacters($value)
    {
        return preg_match('/(\d{3})|([a-zA-Z])\2{2,}/', $value);
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must meet the password policy requirements.';
    }
}

```
4. Copy the `PasswordPolicyRuleTest.php` file from the `tests/Unit/Rules` directory of this repository and paste it into your Laravel project's `tests/Unit/Rules` directory.
```
<?php

namespace Tests\Unit\Rules;

use App\Rules\PasswordPolicyRule;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\ValidationException;
use Tests\TestCase;

class PasswordPolicyRuleTest extends TestCase
{
    /**
     * Test valid passwords that meet the password policy.
     *
     * @return void
     */
    public function testValidPasswords()
    {
        $rule = new PasswordPolicyRule();

        $this->assertTrue($rule->passes('password', 'StrongPass123!'));
        $this->assertTrue($rule->passes('password', 'AnotherValid1@'));
    }

    /**
     * Test invalid passwords that do not meet the password policy.
     *
     * @return void
     */
    public function testInvalidPasswords()
    {
        $rule = new PasswordPolicyRule();

        $this->assertFalse($rule->passes('password', 'short'));
        $this->assertFalse($rule->passes('password', 'alllowercase'));
        $this->assertFalse($rule->passes('password', 'ALLUPPERCASE'));
        $this->assertFalse($rule->passes('password', '12345678'));
        $this->assertFalse($rule->passes('password', '!@#'));
        $this->assertFalse($rule->passes('password', 'Invalid123'));
        $this->assertFalse($rule->passes('password', 'Invalid123@username'));
        $this->assertFalse($rule->passes('password', 'Abc123'));
    }

    /**
     * Test validation using Laravel's Validator with the PasswordPolicyRule.
     *
     * @return void
     */
    public function testValidationUsingValidator()
    {
        $data = ['password' => 'Weak123'];

        $validator = Validator::make($data, [
            'password' => ['required', new PasswordPolicyRule()],
        ]);

        $this->assertTrue($validator->fails());

        $this->expectException(ValidationException::class);
        $validator->validate();
    }
}
```
## Usage

### Creating the PasswordPolicyRule

1. In your Laravel project, navigate to the `app/Rules` directory.

2. Create a new file named `PasswordPolicyRule.php`.

3. Open the file and paste the content of the `PasswordPolicyRule.php` file from this repository into it.

### Writing Unit Tests

1. In your Laravel project, navigate to the `tests/Unit/Rules` directory.

2. Create a new file named `PasswordPolicyRuleTest.php`.

3. Open the file and paste the content of the `PasswordPolicyRuleTest.php` file from this repository into it.

### Using the PasswordPolicyRule

To use the `PasswordPolicyRule` for password validation in your Laravel application, follow these steps:

1. In your controller or form request, add the following use statement:

```php
use App\Rules\PasswordPolicyRule;

$request->validate([
    'password' => ['required', 'string', new PasswordPolicyRule(10)], // Minimum length of 10 characters
]);

```
### Running the Unit Tests
To run the unit tests for the PasswordPolicyRule, follow these steps:

Open your terminal and navigate to your Laravel project's root directory.

Run the PHPUnit tests using the following command:
```
php artisan test --filter PasswordPolicyRuleTest
```

