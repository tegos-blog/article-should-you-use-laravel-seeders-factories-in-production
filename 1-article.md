
## Introduction

When you need to populate your database with test data, Laravel's features on seeding and factories are helpful.
Developers have much debate on whether these characteristics should be used in production environments.
To assist you in selecting the perfect solution for your projects, this post discusses both sides of the argument.

## The Argument Opposing Production in Factories

### Joel Clermont Source

[Joel Clermont](https://masteringlaravel.io/daily/2025-04-01-why-i-would-not-use-a-factory-in-application-code) made a case strongly against the use of factories in application code that executes in production. While factories are great for testing and local dev environments, they were not built for production.

His primary issues are:

1. **Security issues**: Factories sidestep Eloquent's important mass assignment protection.

2. **Unintended fake data**: Factories often utilize the Faker library to generate random values for fields you don't explicitly defined, and might include bogus information into your production database.

3. **Unexpected connections**: Factory relations can create completely fake associated models you didn't intend to create.

Clermont concludes that despite the convenience and code reusability factories might offer, they belong exclusively in testing and seeding, never in production application code.

## The Argument for Not Using Seeders in Production

### The View of Nuno Maduro's

Regarding seeders, [Nuno Maduro](https://www.youtube.com/watch?v=HNgDhZYg3VI) is even more radical. In production pointing to several problems:

1. **Problems with maintenance**: Seeders can become obsolete and stop to accurately depict the current status of your production data.

2. **Inconsistent across environments**: This can be a problem when you need to re-deploy your application to another place.

3. **Model limits**: When utilizing it in seeders and it, you have business logic, events, mutators attached to a model could vary over time for random events to go amiss.

Maduro advise that using migrations will be more efficient than seeders to insert standard production data. For example, when you need to populate a table with unchanging values like user sorts, migration files or the DB facade is preferable route of sql execution.

```php
// In migration file
public function up()
{
    // create user_types table
    // insert user types
    DB::table('user_types')->insert([
        ['name' => 'Admin'],
        ['name' => 'User'],
        ['name' => 'Guest']
    ]);
}
```

Maduro points out that this technique ensures that the information remains constant even if your software changes with time.

## Middle Way - Local Environment Seeders

### How Stefan Zweifel approaches it

[Stefan Zweifel](https://stefanzweifel.dev/posts/2023/01/30/local-environment-seeders-in-laravel/) has a unique take on seeders. He believes many users see them as just tools for development rather than something for production. He introduces the idea of Local Environment Seeders - these are meant specifically for use while you're developing.

This method has a few advantages:

1. **More Simple UI Testing**: The program creates fictitious data so you can see how your application performs in varied circumstances.

2. **New team members** can set up a working environment fast, without having to deal with messy database dumps.

3. **Testing Edge Cases**: It can create users with various statuses (like trial, subscribed, expired) to help you test different scenarios.

By creating a `LocalEnvironmentSeeder` that operates only when you are in a local environment, Zweifel puts this into action:

```php
// DatabaseSeeder.php
public function run()
{
   if (app()->environment('local')) {
    $this->call(LocalEnvironmentSeeder::class);
   }
}
```

## Tips for Using Seeders

These are some valuable advice to remember if you intend to use seeders:

1. **Consistent use enums**:
   ```php
   final class CurrencySeeder extends Seeder
   {
       public function run(): void
       {
           $currencies = [
               [
                   'id' => CurrencyEnum::USD->value,
                   'currency' => CurrencyEnum::USD->name,
                   'symbol' => CurrencyEnum::USD->symbol(),
               ],
               [
                   'id' => CurrencyEnum::UAH->value,
                   'currency' => CurrencyEnum::UAH->name,
                   'symbol' => CurrencyEnum::UAH->symbol(),
               ],
           ];
           
           DB::table('currencies')->insertOrIgnore($currencies);
       }
   }
   ```

2. **Don't use models directly**: remain with the DB interface and table names.

3. **Choose insert methods that handle duplicates**: Use methods like `upsert()` to make reseeding less difficult.

4. **Modular thinking**: Plan your seeders so that when required they can be smoothly shifted to various modules or package.

## Conclusion

From what experienced Laravel developers say, here's the takeaway:

- **Factories** shouldn't be used in production code.
- **Seeders** are mainly for development environments.
- **Migrations** are the way to go for adding static data in production.

Think about what fits your needs and how it will affect your app's maintainability over time.
Seeders might seem easier at first, but using migrations for production data usually leads to more stability as your app grows.
No matter what you pick, keeping your data practices clean and consistent will help your app stay reliable down the road.