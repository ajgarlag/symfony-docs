.. index::
    single: Security; How to control Session Concurrency

How to Control Session Concurrency
==================================

Sometimes, it's useful to restrict the number of concurrent sessions that a given
user can open using different browsers and devices. When the maximum number of
concurrent sessions is achieved, Symfony allows you to disable new sessions or to
expire the old ones. To do so, add a ``session_registry`` section in your security
configuration to save some sessions metadata, and set the ``max_sessions`` limit
using the ``session_concurrency`` option of your firewall:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            session_registry: ~
            firewalls:
                main:
                    # ...
                    session_concurrency:
                        max_sessions: 2

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">
            <config>
                <session-registry />
                <firewall name="main">
                    <!-- ... -->
                    <session-concurrency
                        max-sessions="2"
                    />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'session_registry' => array(),
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'session_concurrency' => array(
                        'max_sessions' => 2,
                    ),
                ),
            ),
        ));

With this configuration, any user will be allowed to open up to 2 sessions, but
will fail to open the third one.

.. note::

    The default storage for the session registry is provided by the Doctrine
    Bridge using the default DBAL connection. This storage can be overriden with
    a custom service implementing the ``SessionRegistryStorageInterface`` and
    setting the ``session_registry_storage`` option to your custom service ID
    inside the ``session_registry`` section.

If you use the default session registry storage, provided by the Doctrine Bridge,
you have to import the database structure. Fortunately, there is a task for this.
Simply run the following command:

.. code-block:: bash

    $ php app/console init:session-registry-storage


.. tip::

    The default session registry storage can be overriden with a custom service
    implementing the ``SessionRegistryStorageInterface`` and setting the
    ``session_registry_storage`` option to your custom service ID inside the
    ``session_registry`` section.

Maybe, you would like to mark older active sessions as expired instead of
disabling the ability to open a new one. This can be achived setting the
``error_if_maximum_exceeded`` option to false in the firewall configuration.

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            session_registry: ~
            firewalls:
                main:
                    # ...
                    session_concurrency:
                        max_sessions: 2
                        error_if_maximum_exceeded: false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">
            <config>
                <session-registry />
                <firewall name="main">
                    <!-- ... -->
                    <session-concurrency
                        max-sessions="2"
                        error-if-maximum-exceeded="false"
                    />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'session_registry' => array(),
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'session_concurrency' => array(
                        'max_sessions' => 2,
                        'error_if_maximum_exceeded' => false,
                    ),
                ),
            ),
        ));

.. caution::

    This option will mark the session as expired in the session registry, but
    it will be necessary to add a listener to your app to prevent the use of
    these expired sessions.

By default, when session concurrency is active in a firewall context, all
sessions will be registered. The automatic session registration can be disabled
setting the ``register_new_sessions`` option to false:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            session_registry: ~
            firewalls:
                main:
                    # ...
                    session_concurrency:
                        max_sessions: 2
                        register_new_sessions: false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">
            <config>
                <session-registry />
                <firewall name="main">
                    <!-- ... -->
                    <session-concurrency
                        max-sessions="2"
                        register-new-sessions="false"
                    />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'session_registry' => array(),
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'session_concurrency' => array(
                        'max_sessions' => 2,
                        'register_new_sessions' => false,
                    ),
                ),
            ),
        ));

If the ``max_sessions`` option is left to its default value (``0``) the maximum
number of sessions will not be checked, but it will allow you to manually expire
all sessions for a concrete user through the session registry:

.. code-block:: php

    // src/Acme/DemoBundle/Controller/DefaultController.php
    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\Security\Core\User\UserInterface;

    class DefaultController extends Controller
    {
        public function expireUserSessionsAction(UserInterface $user)
        {
            /** @var $sessionRegistry \Symfony\Component\Security\Http\Session\SessionRegistry */
            $sessionRegistry = $this->get('security.authentication.session_registry');

            $sessions = $sessionRegistry->getAllSessions($user->getUsername());
            foreach ($sessions as $session) {
                $sessionRegistry->expireNow($session->getSessionId());
            }
        }
    }
