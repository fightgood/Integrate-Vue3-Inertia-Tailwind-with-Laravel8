# Integrate-Vue-Inertia-Tailwind-with-Laravel8
> **NOTICE:** This guide was written using Laravel 8. It also works with Laravel 9 that's using Laravel Mix. If you're using Laravel 9 with Vite, follow this tutorial.

Laravel is by far the most popular open source PHP framework out there. I've been using it since version 4 and today we celebrate the launch of the 9th version. What an achievement!

The beauty of this php framework is not only the ease of writing code but the community behind it which always find new ways to improve the code, develops new packages and also pushes the integration with other awesome frameworks.

For example, if it wasn't for Laravel's creator, Taylor Otwell, I think Vue wouldn't have been so popular today. He stated in a tweet many years ago that Vue was actually easier to learn compared to React... and I couldn't agree more. So even today, even if Laravel has scaffoldings for both JS frameworks, I'd always pick Vue over React, just because it is easier.

So what I'm trying to point out is that Laravel will always try to adopt and support new cool JS frameworks or any other tool that's really a gamechanger. Inertia.js and Tailwind CSS are just two more tools that were added to the book that are really mind blowing.

Before we dive in deep, we just want to make sure we have all the tools we need. We'll be using PHP 8, so make sure you have that installed, Composer and NPM. I'll briefly go over how to install Composer and NPM.

[](#installing-laravel)1. Installing Laravel
-----------------------------------------

To install Laravel you can use Laravel Sail, which will boot up a Docker container, or you can use the old fashioned Laravel installer. I'm using Windows 11 + WSL2 running Ubuntu and I prefer the Laravel installer, so I need to run the following commands one by one. Please note that I'm using Laravel 8 and PHP 8.0.

Making sure we are in the desired folder we're going to require the Laravel's installer globally and then use it to create a new app called "**awesome-app**" (this will automatically create the folder with the same name).  

    composer create-project laravel/laravel laravel8-vue3-inertia-tailwind
    cd laravel8-vue3-inertia-tailwind
    npm install #installs all the dependencies
    
If the `laravel new awesome-app` returns `laravel: command not found` make sure that you have you're Composer's vendor `bin` directory in `$PATH`.

**Now that we have our fresh install, we can go ahead and add Inertia.js, Vue.js and Tailwind CSS.**

[](#installing-tailwind-css)2. Installing Tailwind CSS
---------------------------------------------------

Tailwind requires the least amount of effort. We just need to install `postcss` and `autoprefixer` too. 

    npm install -D tailwindcss postcss autoprefixer
    
Let's create the tailwind config file...  

    npx tailwindcss init
    
...and add our template files so the Tailwind's JIT will know exactly what classes we use in our templates and generate them. So open `tailwind.config.js` and add the following line `./resources/js/**/*.{vue,js}` to the `content` so the file will look like this:  

    module.exports = {
        content: ["./resources/js/**/*.{vue,js}"],
        theme: {
            extend: {},
        },
        plugins: [],
    };

We also have to add the Tailwind's directives to `resources/css/app.css`:  

    @tailwind base;
    @tailwind components;
    @tailwind utilities;

The last thing to do is to require Tailwind into `webpack.mix.js` which uses Laravel Mix to build our assets. We'll get back to our webpack config file later, but for now it will have to look like this:  

    const mix = require("laravel-mix");
    
    mix.js("resources/js/app.js", "public/js").postCss(
        "resources/css/app.css",
        "public/css",
        [require("tailwindcss")]
    );

[](#installing-vuejs)3. Installing Vue.js
--------------------------------------

We'll be using version 3 of Vue. Please note that as of 7th of February 2022, the version 3 has become the default version.

Vue 3 has two different API styles. It still supports the Options API from Vue 2, but it makes available the Composition API. If you don't understand what that is, you can read this short introduction. For the sake of simplicity, we'll be using the Options API since most of the developers are already used to it and have been using it since forever.

So let's add Vue 3:  

    npm install vue@next
    
[](#installing-inertiajs)4. Installing Inertia.js
----------------------------------------------

First we'll need to install Inertia's server side package:  

    composer require inertiajs/inertia-laravel
    
Next we'll need to create the Inertia middleware which handles the requests and also helps us to share data with all our Vue views, similar to `View::share()`.  

    php artisan inertia:middleware
    
`HandleInertiaRequests.php` will be created inside `app/Http/Middleware`. We'll just need to add this middleware to the `web` middleware group inside `app/Http/Kernel.php`:  

    'web' => [
        // ...
        \App\Http\Middleware\HandleInertiaRequests::class,
    ],

Coming next is the Inertia's client side. We're using Vue 3, so we'll install Inertia alongside with the Vue 3 adapter:  

    npm install @inertiajs/inertia @inertiajs/inertia-vue3

Let's throw in the Inertia's progress bar. This will be used as a loading indicator between page navigation.  

    npm install @inertiajs/progress

Inertia uses Laravel's routes, so we won't need to use a client side router, but to make use of Laravel's `web.php` routes, we have to pass them to the DOM somehow. The easiest way to do it to use Ziggy.  
Let's install Ziggy:  

    composer require tightenco/ziggy

Now we can use the `@routes` blade directive inside our blade template to expose the `web.php` routes to the client side.

[](#gluing-everything-together)5. Gluing everything together
---------------------------------------------------------

Now we have everything installed and ready to be used. We have installed **Vue 3**, **Inertia** and **Tailwind CSS**.

Let's start by setting up our one and only **blade** template. We're going to rename the `welcome.blade.php` to `app.blade.php` inside `resources/views`. We're also going to remove all its content and replace it with the following:  

    <!DOCTYPE html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
    
            @routes
            <link href="{{ asset(mix('css/app.css')) }}" rel="stylesheet">
            <script src="{{ asset(mix('js/manifest.js')) }}" defer></script>
            <script src="{{ asset(mix('js/vendor.js')) }}" defer></script>
            <script src="{{ asset(mix('js/app.js')) }}" defer></script>
            <title></title>
            @inertiaHead
        </head>
    
        <body>
            @inertia
        </body>
    
    </html>

So first of all you will notice we don't have any `<title>`. This is because we need it to be dynamic and we can set that using Inertia's `<Head>` component. That's why you can see that we've also added the `@inertiaHead` directive.

We have added the `@routes` directive to pass the Laravel's routes in the document's `<head>`.

We are importing our `app.css` and also a bunch of `.js` we are going to take care shortly.

In the `<body>` we only use the `@inertia` directive which renders a `div` element with a bunch of data passed to it using a `data-page` attribute.

[](#ziggy-setup)6. Ziggy Setup
--------------------------------------

Let's get back to Ziggy and generate the `.js` file that contains all of our routes. We'll gonna import this into our `app.js` a bit later.  

    php artisan ziggy:generate resources/js/ziggy.js

To resolve `ziggy` in Vue, we'll have to add an alias to the Vue driver in `webpack.mix.js`:  

    const path = require("path");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });

[](#setting-up-appjs)7. Setting up app.js
--------------------------------------

Let's move on by setting up our app.js file. This is our main main file we're going to load in our blade template.

> You can delete `bootstrap.js` from `resources/js`. That's no longer needed.

Now open `resources/js/app.js` and delete everything from it and add the following chunk of code:  

    import { createApp, h } from "vue";
    import { createInertiaApp, Link, Head } from "@inertiajs/inertia-vue3";
    import { InertiaProgress } from "@inertiajs/progress";
    
    import { ZiggyVue } from "ziggy";
    import { Ziggy } from "./ziggy";
    
    InertiaProgress.init();
    
    createInertiaApp({
        resolve: async (name) => {
            return (await import(`./Pages/${name}`)).default;
        },
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
                .use(plugin)
                .use(ZiggyVue, Ziggy)
                .component("Link", Link)
                .component("Head", Head)
                .mixin({ methods: { route } })
                .mount(el);
        },
    });

What does this is to import Vue, Inertia, Inertia Progress and Ziggy and then create the Inertia App. We're also passing the `Link` and `Head` components as globals because we're going to use them a lot.

Inertia will load our pages from the `Pages` directory so I'm gonna create 3 demo pages in that folder. Like so:

![demo-inertia-pages](https://res.cloudinary.com/practicaldev/image/fetch/s--jUjLsSuA--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m5li6dw9pqiawrbp76di.png)

Each page will container the following template. The `Homepage` text will be replaced based on the file's name:  

    <template>
        <h1>Homepage</h1>
    </template>

The next step is to add the missing pieces to the `webpack.mix.js` file. Everything needs to look like this:  

    const path = require("path");
    const mix = require("laravel-mix");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });
    
    // Build files
    mix.js("resources/js/app.js", "public/js")
        .vue({ version: 3 })
        .webpackConfig({
            resolve: {
                alias: {
                    "@": path.resolve(__dirname, "resources/js"),
                },
            },
        })
        .extract()
        .postCss("resources/css/app.css", "public/css", [require("tailwindcss")])
        .version();

You can see that we're specifying the Vue version that we're using, we're also setting and alias (`@`) for our root js path and we're also using `.extract()` to split our code into smaller chunks (optional, but better for production in some use cases).

[](#setting-up-our-laravel-routes)8. Setting up our Laravel routes
---------------------------------------------------------------

We've taken care of almost everything. Not we just need to create routes for each of the Vue pages we have created.

Let's open the `routes/web.php` file and replace everything there with the following:  

    <?php
    
    use Illuminate\Support\Facades\Route;
    use Inertia\Inertia;
    
    Route::get(
        '/',
        static function () {
            return Inertia::render(
                'Home',
                [
                    'title' => 'Homepage',
                ]
            );
        }
    )->name('homepage');
    
    Route::get(
        '/about',
        static function () {
            return Inertia::render(
                'About',
                [
                    'title' => 'About',
                ]
            );
        }
    )->name('about');
    
    Route::get(
        '/contact',
        static function () {
            return Inertia::render(
                'Contact',
                [
                    'title' => 'Contact',
                ]
            );
        }
    )->name('contact');

You can notice right away that we're not returning any traditional blade view. Instead we return an `Inertia::render()` response which takes 2 parameters. The first parameter is the name of our Vue page and the 2nd is an array of properties that will be passed to the Vue page using `$page.props`.

[](#modifying-the-vue-pages)9. Modifying the Vue pages
---------------------------------------------------

Knowing this we can modify our pages to the following template and also add a navigation to them:  

    <template>
        <Head>
            <title>{{ $page.props.title }} - My awesome app</title>
        </Head>
    
        <div class="p-6">
            <div class="flex space-x-4 mb-4">
                <Link
                    :href="route('homepage')"
                    class="text-gray-700 bg-gray-200 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
                    >Homepage</Link
                >
                <Link
                    :href="route('about')"
                    class="text-gray-700 bg-gray-200 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
                    >About</Link
                >
                <Link
                    :href="route('contact')"
                    class="text-gray-700 bg-gray-200 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"
                    >Contact</Link
                >
            </div>
    
            <h1>This is: {{ $page.props.title }}</h1>
        </div>
    </template>

Now we have a simple navigation on each page and also a dynamic page `<title>`. The only thing left now is to compile everything and start the server:  

    npm install
    npm update vue-loader
    npm run dev
    php artisan serve

The last command will start a server on your localhost using the 8000 port `http://127.0.0.1:8000/` so navigating to it you will be able to see the final result.

[](#testing)10. Testing
-------------------

It should look similar to this:  
![inertia-demo.gif](https://res.cloudinary.com/practicaldev/image/fetch/s--cKk3RI2D--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://i.imgur.com/WRIN3Ec.gif)

That's pretty much everything you need to know. Of course there's more like using Laravel's lang files, Vue layouts, server side rendering... but maybe in a part 2.

Enjoy!

Credit/Reference : https://dev.to/geowrgetudor/setting-up-laravel-with-inertiajs-vuejs-tailwind-css-21pc
