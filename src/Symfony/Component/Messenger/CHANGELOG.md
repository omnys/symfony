CHANGELOG
=========

4.4.0
-----

 * Deprecated passing a `ContainerInterface` instance as first argument of the `ConsumeMessagesCommand` constructor,
   pass a `RoutableMessageBus`  instance instead.
 * Added support for auto trimming of Redis streams.
 * `InMemoryTransport` handle acknowledged and rejected messages.

4.3.0
-----

 * Added `NonSendableStampInterface` that a stamp can implement if
   it should not be sent to a transport. Transport serializers
   must now check for these stamps and not encode them.
 * [BC BREAK] `SendersLocatorInterface` has an additional method:
   `getSenderByAlias()`.
 * Removed argument `?bool &$handle = false` from `SendersLocatorInterface::getSenders`
 * A new `ListableReceiverInterface` was added, which a receiver
   can implement (when applicable) to enable listing and fetching
   individual messages by id (used in the new "Failed Messages" commands).
 * Both `SenderInterface::send()` and `ReceiverInterface::get()`
   should now (when applicable) add a `TransportMessageIdStamp`.
 * Added `WorkerStoppedEvent` dispatched when a worker is stopped.
 * Added optional `MessageCountAwareInterface` that receivers can implement
   to give information about how many messages are waiting to be processed.
 * [BC BREAK] The `Envelope::__construct()` signature changed:
   you can no longer pass an unlimited number of stamps as the second,
   third, fourth, arguments etc: stamps are now an array passed to the
   second argument.
 * [BC BREAK] The `MessageBusInterface::dispatch()` signature changed:
   a second argument `array $stamps = []` was added.
 * Added new `messenger:stop-workers` command that sends a signal
   to stop all `messenger:consume` workers.
 * [BC BREAK] The `TransportFactoryInterface::createTransport()` signature
   changed: a required 3rd `SerializerInterface` argument was added.
 * Added a new `SyncTransport` to explicitly handle messages synchronously.
 * Added `AmqpStamp` allowing to provide a routing key, flags and attributes on message publishing.
 * [BC BREAK] Removed publishing with a `routing_key` option from queue configuration, for
   AMQP. Use exchange `default_publish_routing_key` or `AmqpStamp` instead.
 * [BC BREAK] Changed the `queue` option in the AMQP transport DSN to be `queues[name]`. You can 
   therefore name the queue but also configure `binding_keys`, `flags` and `arguments`.
 * [BC BREAK] The methods `get`, `ack`, `nack` and `queue` of the AMQP `Connection` 
   have a new argument: the queue name.
 * Added optional parameter `prefetch_count` in connection configuration, 
   to setup channel prefetch count.
 * New classes: `RoutableMessageBus`, `AddBusNameStampMiddleware`
   and `BusNameStamp` were added, which allow you to add a bus identifier
   to the `Envelope` then find the correct bus when receiving from
   the transport. See `ConsumeMessagesCommand`.
 * The optional `$busNames` constructor argument of the class `ConsumeMessagesCommand` was removed.
 * [BC BREAK] 3 new methods were added to `ReceiverInterface`:
   `ack()`, `reject()` and `get()`. The methods `receive()`
   and `stop()` were removed.
 * [BC BREAK] Error handling was moved from the receivers into
   `Worker`. Implementations of `ReceiverInterface::handle()`
   should now allow all exceptions to be thrown, except for transport
   exceptions. They should also not retry (e.g. if there's a queue,
   remove from the queue) if there is a problem decoding the message.
 * [BC BREAK] `RejectMessageExceptionInterface` was removed and replaced
   by `Symfony\Component\Messenger\Exception\UnrecoverableMessageHandlingException`,
   which has the same behavior: a message will not be retried
 * The default command name for `ConsumeMessagesCommand` was
   changed from `messenger:consume-messages` to `messenger:consume`
 * `ConsumeMessagesCommand` has two new optional constructor arguments
 * [BC BREAK] The first argument to Worker changed from a single
   `ReceiverInterface` to an array of `ReceiverInterface`.
 * `Worker` has 3 new optional constructor arguments.
 * The `Worker` class now handles calling `pcntl_signal_dispatch()` the
   receiver no longer needs to call this.
 * The `AmqpSender` will now retry messages using a dead-letter exchange
   and delayed queues, instead of retrying via `nack()`
 * Senders now receive the `Envelope` with the `SentStamp` on it. Previously,
   the `Envelope` was passed to the sender and *then* the `SentStamp`
   was added.
 * `SerializerInterface` implementations should now throw a
   `Symfony\Component\Messenger\Exception\MessageDecodingFailedException`
   if `decode()` fails for any reason.
 * [BC BREAK] The default `Serializer` will now throw a
   `MessageDecodingFailedException` if `decode()` fails, instead
   of the underlying exceptions from the Serializer component.
 * Added `PhpSerializer` which uses PHP's native `serialize()` and
   `unserialize()` to serialize messages to a transport
 * [BC BREAK] If no serializer were passed, the default serializer
   changed from `Serializer` to `PhpSerializer` inside `AmqpReceiver`,
   `AmqpSender`, `AmqpTransport` and `AmqpTransportFactory`.
 * Added `TransportException` to mark an exception transport-related
 * [BC BREAK] If listening to exceptions while using `AmqpSender` or `AmqpReceiver`, `\AMQPException` is
   no longer thrown in favor of `TransportException`.
 * Deprecated `LoggingMiddleware`, pass a logger to `SendMessageMiddleware` instead.
 * [BC BREAK] `Connection::__construct()` and `Connection::fromDsn()`
   both no longer have `$isDebug` arguments.
 * [BC BREAK] The Amqp Transport now automatically sets up the exchanges
   and queues by default. Previously, this was done when in "debug" mode
   only. Pass the `auto_setup` connection option to control this.
 * Added a `SetupTransportsCommand` command to setup the transports
 * Added a Doctrine transport. For example, use the `doctrine://default` DSN (this uses the `default` Doctrine entity manager)
 * [BC BREAK] The `getConnectionConfiguration` method on Amqp's `Connection` has been removed. 
 * [BC BREAK] A `HandlerFailedException` exception will be thrown if one or more handler fails.
 * [BC BREAK] The `HandlersLocationInterface::getHandlers` method needs to return `HandlerDescriptor`
   instances instead of callables.
 * [BC BREAK] The `HandledStamp` stamp has changed: `handlerAlias` has been renamed to `handlerName`,
   `getCallableName` has been removed and its constructor only has 2 arguments now.
 * [BC BREAK] The `ReceivedStamp` needs to exposes the name of the transport from which the message
   has been received.

4.2.0
-----

 * Added `HandleTrait` leveraging a message bus instance to return a single 
   synchronous message handling result
 * Added `HandledStamp` & `SentStamp` stamps
 * All the changes below are BC BREAKS
 * Senders and handlers subscribing to parent interfaces now receive *all* matching messages, wildcard included
 * `MessageBusInterface::dispatch()`, `MiddlewareInterface::handle()` and `SenderInterface::send()` return `Envelope`
 * `MiddlewareInterface::handle()` now require an `Envelope` as first argument and a `StackInterface` as second
 * `EnvelopeAwareInterface` has been removed
 * The signature of `Amqp*` classes changed to take a `Connection` as a first argument and an optional
   `Serializer` as a second argument.
 * `MessageSubscriberInterface::getHandledMessages()` return value has changed. The value of an array item
   needs to be an associative array or the method name.
 * `StampInterface` replaces `EnvelopeItemInterface` and doesn't extend `Serializable` anymore
 * The `ConsumeMessagesCommand` class now takes an instance of `Psr\Container\ContainerInterface`
   as first constructor argument
 * The `EncoderInterface` and `DecoderInterface` have been replaced by a unified `Symfony\Component\Messenger\Transport\Serialization\SerializerInterface`.
 * Renamed `EnvelopeItemInterface` to `StampInterface`
 * `Envelope`'s constructor and `with()` method now accept `StampInterface` objects as variadic parameters
 * Renamed and moved `ReceivedMessage`, `ValidationConfiguration` and `SerializerConfiguration` in the `Stamp` namespace
 * Removed the `WrapIntoReceivedMessage` class
 * `MessengerDataCollector::getMessages()` returns an iterable, not just an array anymore
 * `HandlerLocatorInterface::resolve()` has been removed, use `HandlersLocator::getHandlers()` instead
 * `SenderLocatorInterface::getSenderForMessage()` has been removed, use `SendersLocator::getSenders()` instead
 * Classes in the `Middleware\Enhancers` sub-namespace have been moved to the `Middleware` one
 * Classes in the `Asynchronous\Routing` sub-namespace have been moved to the `Transport\Sender\Locator` sub-namespace
 * The `Asynchronous/Middleware/SendMessageMiddleware` class has been moved to the `Middleware` namespace
 * `SenderInterface` has been moved to the `Transport\Sender` sub-namespace
 * The `ChainHandler` and `ChainSender` classes have been removed
 * `ReceiverInterface` and its implementations have been moved to the `Transport\Receiver` sub-namespace
 * `ActivationMiddlewareDecorator` has been renamed `ActivationMiddleware`
 * `AllowNoHandlerMiddleware` has been removed in favor of a new constructor argument on `HandleMessageMiddleware`
 * The `ContainerHandlerLocator`, `AbstractHandlerLocator`, `SenderLocator` and `AbstractSenderLocator` classes have been removed
 * `Envelope::all()` takes a new optional `$stampFqcn` argument and returns the stamps for the specified FQCN, or all stamps by their class name
 * `Envelope::get()` has been renamed `Envelope::last()`

4.1.0
-----

 * Introduced the component as experimental
