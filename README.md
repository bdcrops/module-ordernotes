# Magento2x OrderNotes

This module is used as a OrderNotes for Magento 2 extensions.
we have built a small, but functional, order notes module. This allowed us to
familiarize ourselves with an important aspect of customizing the checkout experience. The gist
of this lies in understanding the checkout_index_index layout handle, the JavaScript
window.checkoutConfig object, and the uiComponent.


## Goal

- Customizing & Passing data Checkout Experiences
- Adding order notes to the checkout
- Learn Magento 2 Certified Professional Developer exam topics "Customizing the Checkout Process 13%"

## 1. Install & upgrade  OrderNotes


#### 1.1 Copy and paste

If you don't want to install via composer, you can use this way.

- Download [the latest version here](https://github.com/bdcrops/module-ordernotes/archive/master.zip)
- Extract `master.zip` file to `app/code/BDC/OrderNotes` ; You should create a folder path `app/code/BDC/OrderNotes` if not exist.
- Go to Magento root folder and run upgrade command line to install `BDC_OrderNotes`:

```
php bin/magento setup:upgrade
php bin/magento setup:static-content:deploy
```


#### 1.2 Install via composer

We recommend you to install BDC_OrderNotes module via composer. It is easy to install, update and maintaince.Run the following command in Magento 2 root folder.

```
composer config repositories.module-ordernotes git
https://github.com/bdcrops/module-ordernotes.git

composer require bdcrops/module-ordernotes:~1.0.0
php bin/magento setup:upgrade
php bin/magento setup:static-content:deploy
```

#### 1.3 Upgrade    

```
composer update bdcrops/module-ordernotes
php bin/magento setup:upgrade
php bin/magento setup:static-content:deploy
```

Run compile if your store in Product mode:

```
php bin/magento setup:di:compile
```

## 2. Magento 2 Module "OrderNotes" Step By Step Tutorial

- Create app/code/BDC/OrderNotes/registration.php
```
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'BDC_OrderNotes',
    __DIR__
);
```
- Create app/code/BDC/OrderNotes/etc/module.xml
```
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="BDC_OrderNotes" setup_version="1.0.0"/>
</config>
```
- Create app/code/BDC/OrderNotes/Setup/InstallSchema.php
```
<?php

namespace BDC\OrderNotes\Setup;
use \Magento\Framework\Setup\InstallSchemaInterface;

class InstallSchema implements InstallSchemaInterface{
    public function install(
        \Magento\Framework\Setup\SchemaSetupInterface $setup,
        \Magento\Framework\Setup\ModuleContextInterface $context
    ) {
        $connection = $setup->getConnection();
        $connection->addColumn(
            $setup->getTable('quote'),
            'order_notes', [
                'type' => \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
                'nullable' => true,
                'comment' => 'Order Notes'
            ]
        );

        $connection->addColumn(
            $setup->getTable('sales_order'),
            'order_notes', [
                'type' => \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
                'nullable' => true,
                'comment' => 'Order Notes'
            ]
        );
    }
}

```
- Create app/code/BDC/OrderNotes/etc/frontend/routes.xml
```
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="ordernotes" frontName="ordernotes">
            <module name="BDC_OrderNotes"/>
        </route>
    </router>
</config>

```
- Create app/code/BDC/OrderNotes/Controller/Index.php
```
<?php
namespace BDC\OrderNotes\Controller;
abstract class Index extends \Magento\Framework\App\Action\Action { }

```
- Create app/code/BDC/OrderNotes/Controller/Index/Process.php
```
<?php
namespace BDC\OrderNotes\Controller\Index;

class Process extends \BDC\OrderNotes\Controller\Index {
    protected $checkoutSession;
    protected $logger;

    public function __construct(
        \Magento\Framework\App\Action\Context $context,
        \Magento\Checkout\Model\Session $checkoutSession,
        \Psr\Log\LoggerInterface $logger ) {
        $this->checkoutSession = $checkoutSession;
        $this->logger = $logger;
        parent::__construct($context);
    }
    public function execute() {
        try {
            if ($notes = $this->getRequest()->getParam('order_notes', null)) {
                $quote = $this->checkoutSession->getQuote();
                $quote->setOrderNotes($notes);
                $quote->save();
            }
            $result = ['time' => (new \DateTime('now'))->format('Y-m-d H:i:s'), ];
        } catch (\Exception $e) {
            $this->logger->critical($e);
            $result = [
                'error' => __('Something went wrong.'),
                'errorcode' => $e->getCode(),
            ];
        }
        $resultJson = $this->resultFactory->create(\Magento\Framework\Controller\ResultFactory::TYPE_JSON);
        $resultJson->setData($result);
        return $resultJson;
    }
}

```

- Create app/code/BDC/OrderNotes/etc/frontend/di.xml
```
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Checkout\Model\CompositeConfigProvider">
        <arguments>
            <argument name="configProviders" xsi:type="array">
                <item name="order_notes_config_provider" xsi:type="object">BDC\OrderNotes\Model\ConfigProvider</item>
            </argument>
        </arguments>
    </type>
</config>
```
- Create app/code/BDC/OrderNotes/Model/ConfigProvider.php
```
<?php
namespace BDC\OrderNotes\Model;

use \Magento\Checkout\Model\ConfigProviderInterface;

class ConfigProvider implements ConfigProviderInterface {
    public function getConfig() {
        return [
            'orderNotes' => [
                'title' => __('Order Notes'),
                'header' => __('Header content.'),
                'footer' => __('Footer content.'),
                'options' => [
                    [ 'code' => 'ring', 'value' => __('Ring longer') ],
                    [ 'code' => 'backyard', 'value' => __('Try backyard') ],
                    [ 'code' => 'neighbour', 'value' => __('Ping neighbour') ],
                    [ 'code' => 'other', 'value' => __('Other') ],
                ],
                'time' => (new \DateTime('now'))->format('Y-m-d H:i:s')
            ]
        ];
    }
}

```
- Create app/code/BDC/OrderNotes/etc/webapi_rest/events.xml
```
<?xml version="1.0"?>

<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Event/etc/events.xsd">
    <event name="sales_model_service_quote_submit_before">
        <observer name="orderNotesToOrder" instance="BDC\OrderNotes\Observer\SaveOrderNotesToOrder" shared="false"/>
    </event>
</config>

```
- Create app/code/BDC/OrderNotes/Observer/SaveOrderNotesToOrder.php
```
<?php
namespace BDC\OrderNotes\Observer;
use \Magento\Framework\Event\ObserverInterface;
use \Magento\Framework\Event\Observer;

class SaveOrderNotesToOrder implements ObserverInterface {
    public function execute(Observer $observer) {
        $event = $observer->getEvent();
        if ($notes = $event->getQuote()->getOrderNotes()) {
            $event->getOrder()->setOrderNotes($notes)
                ->addStatusHistoryComment('Customer note: ' . $notes);
        }
        return $this;
    }
}

```
- Create app/code/BDC/OrderNotes/view/frontend/layout/checkout_index_index.xml
```
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceBlock name="checkout.root">
            <arguments>
                <argument name="jsLayout" xsi:type="array">
                    <item name="components" xsi:type="array">
                        <item name="checkout" xsi:type="array">
                            <item name="children" xsi:type="array">
                                <item name="steps" xsi:type="array">
                                    <item name="children" xsi:type="array">
                                        <item name="order-notes" xsi:type="array">
                                            <item name="component" xsi:type="string">BDC_OrderNotes/js/view/order-notes</item>
                                            <item name="sortOrder" xsi:type="string">2</item>
                                        </item>
                                    </item>
                                </item>
                            </item>
                        </item>
                    </item>
                </argument>
            </arguments>
        </referenceBlock>
    </body>
</page>

```
- Create app/code/BDC/OrderNotes/view/frontend/web/js/view/order-notes.js
```
define( ['ko',
        'uiComponent',
        'underscore',
        'Magento_Checkout/js/model/step-navigator',
        'jquery',
        'mage/translate',
        'mage/url'],
    function (
        ko,
        Component,
        _,
        stepNavigator,
        $,
        $t,
        url ) {
        'use strict';
        let checkoutConfigOrderNotes = window.checkoutConfig.orderNotes;
        return Component.extend({
            defaults: { template: 'BDC_OrderNotes/order/notes' },
            isVisible: ko.observable(true),
            initialize: function () {
                this._super();
                stepNavigator.registerStep('order_notes', null, $t('Order Notes'), this.isVisible, _.bind(this.navigate, this), 15);
                return this;
            },

            getTitle: function () { return checkoutConfigOrderNotes.title; },
            getHeader: function () { return checkoutConfigOrderNotes.header; },
            getFooter: function () { return checkoutConfigOrderNotes.footer; },
            getNotesOptions: function () { return checkoutConfigOrderNotes.options; },
            getCheckoutConfigOrderNotesTime: function ()
              { return checkoutConfigOrderNotes.time;},
            setOrderNotes: function (valObj, event) {
                if (valObj.code == 'other') {
                    $('[name="order_notes"]').val('');
                } else {
                    $('[name="order_notes"]').val(valObj.value);
                }
                return true;
            },

            navigate: function () {
                // Code to trigger when landing on our step
            },

            navigateToNextStep: function () {
                if ($(arguments[0]).is('form')) {
                    $.ajax({
                        type: 'POST',
                        url: url.build('ordernotes/index/process'),
                        data: $(arguments[0]).serialize(),
                        showLoader: true,
                        complete: function (response) {
                            stepNavigator.next();
                        }
                    });
                }
            }
        });
    }
);

```
- Create app/code/BDC/OrderNotes/view/frontend/web/template/order/notes.html
```
<li id="order_notes" data-bind="fadeVisible: isVisible">
    <div class="step-title" data-bind="text: getTitle()" data-role="title"></div>
    <div id="step-content" class="step-content" data-role="content">
        <div class="step-header" data-bind="text: getHeader()" data-role="header"></div>
        <form data-bind="submit: navigateToNextStep" novalidate="novalidate">
            <div data-bind="foreach: getNotesOptions()" class="field choice">
                <input type="radio" name="order[notes]" class="radio"
                       data-bind="value: code, click: $parent.setOrderNotes"/>
                <label data-bind="attr: {'for': code}" class="label">
                    <span data-bind="text: value"></span>
                </label>
            </div>
            <textarea name="order_notes"></textarea>
            <div class="actions-toolbar">
                <div class="primary">
                    <button data-role="opc-continue" type="submit" class="button action continue primary">
                        <span><!-- ko i18n: 'Next'--><!-- /ko --></span>
                    </button>
                </div>
            </div>
        </form>
        <div class="step-footer" data-bind="text: getFooter()" data-role="footer"></div>
    </div>
</li>


```
- OrderNotes Checkout

![](docs/OrderNotesCheckout.png)

-  Order Notes Order Details
![](docs/OrderNotesOrderDetails.png)

- Order Notes Order Comment
![](docs/OrderNotesOrderComent.png)


## 3. FAQ

## Ref
