Rate Limiting Middleware
########################

.. versionadded:: 5.3

The ``RateLimitMiddleware`` provides configurable rate limiting for your
application to protect against abuse and ensure fair usage of resources.

Basic Usage
===========

To use rate limiting in your application, add the middleware to your
middleware queue::

    // In src/Application.php
    use Cake\Http\Middleware\RateLimitMiddleware;

    public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue
    {
        $middlewareQueue
            // ... other middleware
            ->add(new RateLimitMiddleware([
                'limit' => 60,        // 60 requests
                'window' => 60,       // per 60 seconds
                'identifier' => 'ip', // based on IP address
            ]));

        return $middlewareQueue;
    }

When a client exceeds the rate limit, they will receive a
``429 Too Many Requests`` response.

Configuration Options
=====================

The middleware accepts the following configuration options:

- **limit** - Maximum number of requests allowed (default: 60)
- **window** - Time window in seconds (default: 60)
- **identifier** - How to identify clients: 'ip', 'user', 'route', 'api_key' (default: 'ip')
- **strategy** - Rate limiting algorithm: 'sliding_window', 'fixed_window', 'token_bucket' (default: 'sliding_window')
- **cache** - Cache configuration to use (default: 'default')
- **headers** - Whether to include rate limit headers in responses (default: true)
- **message** - Custom error message for rate limit exceeded
- **skipCheck** - Callback to determine if a request should skip rate limiting
- **costCallback** - Callback to determine the cost of a request
- **identifierCallback** - Callback to determine the identifier for a request
- **limitCallback** - Callback to determine the limit for a specific identifier

Identifier Types
================

IP Address
----------

The default identifier type tracks requests by IP address::

    new RateLimitMiddleware([
        'identifier' => 'ip',
        'limit' => 100,
        'window' => 60,
    ])

The middleware automatically handles proxy headers like ``X-Forwarded-For``
and ``CF-Connecting-IP``.

User-based
----------

Track requests per authenticated user::

    new RateLimitMiddleware([
        'identifier' => 'user',
        'limit' => 1000,
        'window' => 3600, // 1 hour
    ])

This requires authentication middleware to be loaded before rate limiting.

Route-based
-----------

Apply different limits to different routes::

    new RateLimitMiddleware([
        'identifier' => 'route',
        'limit' => 10,
        'window' => 60,
    ])

This creates separate limits for each controller/action combination.

API Key
-------

Track requests by API key::

    new RateLimitMiddleware([
        'identifier' => 'api_key',
        'limit' => 5000,
        'window' => 3600,
    ])

The middleware looks for API keys in the ``X-API-Key`` header or
``api_key`` query parameter.

Custom Identifiers
==================

You can create custom identifiers using a callback::

    new RateLimitMiddleware([
        'identifierCallback' => function ($request) {
            // Custom logic to identify the client
            $tenant = $request->getHeader('X-Tenant-ID');
            return 'tenant_' . $tenant[0];
        },
    ])

Rate Limiting Strategies
========================

Sliding Window
--------------

The default strategy that provides smooth rate limiting by continuously
adjusting the window based on request timing::

    new RateLimitMiddleware([
        'strategy' => 'sliding_window',
    ])

Fixed Window
------------

Resets the counter at fixed intervals::

    new RateLimitMiddleware([
        'strategy' => 'fixed_window',
    ])

Token Bucket
------------

Allows for burst capacity while maintaining an average rate::

    new RateLimitMiddleware([
        'strategy' => 'token_bucket',
        'limit' => 100,    // bucket capacity
        'window' => 60,    // refill rate
    ])

Advanced Usage
==============

Skip Rate Limiting
------------------

Skip rate limiting for certain requests::

    new RateLimitMiddleware([
        'skipCheck' => function ($request) {
            // Skip rate limiting for health checks
            return $request->getParam('action') === 'health';
        },
    ])

Request Cost
------------

Assign different costs to different types of requests::

    new RateLimitMiddleware([
        'costCallback' => function ($request) {
            // POST requests cost 5x more
            return $request->getMethod() === 'POST' ? 5 : 1;
        },
    ])

Dynamic Limits
--------------

Set different limits for different users or plans::

    new RateLimitMiddleware([
        'limitCallback' => function ($request, $identifier) {
            $user = $request->getAttribute('identity');
            if ($user && $user->plan === 'premium') {
                return 10000; // Premium users get higher limit
            }
            return 100; // Free tier limit
        },
    ])

Rate Limit Headers
==================

When enabled, the middleware adds the following headers to responses:

- ``X-RateLimit-Limit`` - The maximum number of requests allowed
- ``X-RateLimit-Remaining`` - The number of requests remaining
- ``X-RateLimit-Reset`` - Unix timestamp when the rate limit resets
- ``X-RateLimit-Reset-Date`` - ISO 8601 date when the rate limit resets

Multiple Rate Limiters
======================

You can apply multiple rate limiters with different configurations::

    // Strict limit for login attempts
    $middlewareQueue->add(new RateLimitMiddleware([
        'identifier' => 'ip',
        'limit' => 5,
        'window' => 900, // 15 minutes
        'skipCheck' => function ($request) {
            return $request->getParam('action') !== 'login';
        },
    ]));

    // General API rate limit
    $middlewareQueue->add(new RateLimitMiddleware([
        'identifier' => 'api_key',
        'limit' => 1000,
        'window' => 3600,
    ]));

Cache Configuration
===================

The rate limiter stores its data in cache. Make sure you have a persistent
cache configured::

    // In config/app.php
    'Cache' => [
        'rate_limit' => [
            'className' => 'Redis',
            'prefix' => 'rate_limit_',
            'duration' => '+1 hour',
        ],
    ],

Then use it in the middleware::

    new RateLimitMiddleware([
        'cache' => 'rate_limit',
    ])

.. warning::
    The ``File`` cache engine is not recommended for production use with
    rate limiting as it may not handle concurrent requests properly.