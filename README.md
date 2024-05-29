# План проектов DecimalTools

Добро пожаловать в план проектов по развитию проекта DecimalTools. Ниже представлены подробные описания наших ключевых проектов, направленных на улучшение производительности, безопасности и функциональности экосистемы Decimal.

## Проект 1: DecimalNodeLoadBalancer
### Балансировщик запросов Нод

**Описание:**  
Этот проект направлен на создание системы распределения нагрузки и повышения производительности и отказоустойчивости блокчейна Decimal. Идея заключается в том, чтобы объединить ноды сообщества для эффективного распределения запросов между ними. Участники, предоставляющие свои ноды, будут вносить вклад в общий пул ресурсов, который затем будет использоваться для обработки запросов, распределяя их равномерно и предотвращая перегрузки.

**Основные функции:**
- Автоматическое распределение запросов между нодами.
- Повышение отказоустойчивости за счет резервирования.
- Мониторинг производительности и состояния нод в реальном времени.

## Проект 2: SecureCustodyWallet
### Безопасное Хранение и использование Кастодиальных Кошельков

**Описание:**  
Проект включает разработку микросервиса для безопасного хранения и управления кастодиальными кошельками, используя централизованный узел передачи данных. Сервер, развернутый на вашем оборудовании, будет единственным местом, где создаются и выполняются транзакции. Центральный узел служит гарантом выполнения транзакций, даже если основной сервер временно недоступен. Все транзакции будут храниться в зашифрованном виде, и только вы будете иметь доступ к ключам для расшифровки.

**Основные функции:**
- Безопасное создание и управление кошельками.
- Централизованный узел передачи данных для гарантии выполнения транзакций.
- Хранение транзакций в зашифрованном виде для дополнительной безопасности.
- Открытый исходный код для проверки и улучшения безопасности.

## Проект 3: Decimal Proxy Pay
### Прозрачный Платежный Шлюз

**Описание:**  
Этот проект разрабатывает прозрачный платежный шлюз с открытым API, который позволит быстро интегрировать функционал оплаты счетов в любой монете Decimal в ваши сервисы. Сервис будет генерировать уникальные счета с отдельными кошельками для отслеживания поступлений и отправки callback-уведомлений на ваш сервис. Все поступившие токены будут автоматически перенаправлены на указанный вами кошелек.

**Основные функции:**
- Создание и управление уникальными счетами для оплаты.
- Автоматическое отслеживание поступлений и отправка callback-уведомлений.
- Возможность ручного отслеживания статуса транзакций.
- Автоматическая пересылка поступивших токенов на ваш кошелек.

---

Эти проекты направлены на улучшение производительности, безопасности и функциональности экосистемы Decimal, обеспечивая надежные и удобные инструменты для сообщества.

---

# DecimalTools Project Plan

Welcome to the DecimalTools development project plan. Below you will find detailed descriptions of our key projects aimed at enhancing the performance, security, and functionality of the Decimal ecosystem.

## Project 1: DecimalNodeLoadBalancer
### Node Request Load Balancer

**Description:**  
This project aims to create a system for load distribution to enhance the performance and fault tolerance of the Decimal blockchain. The idea is to combine community nodes to effectively distribute requests among them. Participants providing their nodes will contribute to a common resource pool, which will then be used to handle requests, distributing them evenly and preventing overloads.

**Key Features:**
- Automatic distribution of requests between nodes.
- Increased fault tolerance through redundancy.
- Real-time monitoring of node performance and status.

## Project 2: SecureCustodyWallet
### Secure Custodial Wallet Storage and Management

**Description:**  
This project involves developing a microservice for the secure storage and management of custodial wallets using a centralized data transfer node. A server deployed on your equipment will be the sole location for creating and executing transactions. The central node guarantees transaction execution even if the primary server is temporarily unavailable. All transactions will be stored in encrypted form, and only you will have access to the decryption keys.

**Key Features:**
- Secure creation and management of wallets.
- Centralized data transfer node to ensure transaction execution.
- Encrypted transaction storage for added security.
- Open-source code for security verification and improvement.

## Project 3: Decimal Proxy Pay
### Transparent Payment Gateway

**Description:**  
This project develops a transparent payment gateway with an open API, allowing you to quickly integrate billing functionality in any Decimal currency into your services. The service will generate unique invoices with separate wallets for tracking deposits and sending callback notifications to your service. All incoming tokens will be automatically forwarded to a specified wallet.

**Key Features:**
- Creation and management of unique payment invoices.
- Automatic tracking of deposits and sending callback notifications.
- Manual tracking of transaction status.
- Automatic forwarding of received tokens to your wallet.

---

These projects aim to enhance the performance, security, and functionality of the Decimal ecosystem, providing reliable and convenient tools for the community.

