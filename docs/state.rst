.. _state:

Tracking Conversation State
===========================

The ``DialogueStateTracker`` is the stateful object which keeps track of a conversation. 
The only way the tracker should ever be updated is by passing ``events`` to the ``log_event`` method.
For example:

.. doctest::

    >>> from rasa_core.trackers import DialogueStateTracker
    >>> from rasa_core.slots import TextSlot
    >>> from rasa_core.events import SetSlot

    >>> tracker = DialogueStateTracker("default", slots=[TextSlot("cuisine")])
    >>> print(tracker.slots)
    {u'cuisine': <TextSlot(cuisine: None)>}
    >>> tracker.log_event(SetSlot("cuisine","Mexican"))
    >>> print(tracker.slots)
    {u'cuisine': <TextSlot(cuisine: Mexican)>}


The full set of events is documented in the :doc:`Events API documentation <api/events>`.


Persisting Trackers:
--------------------

When you're running your bot in production,
you want your application to be stateless.
For example, you wouldn't want to lose track of every conversation
each time you restart a running process.
That's why Rasa persists trackers in a key-value store.
For testing, the ``InMemoryTrackerStore`` is sufficient, 
but in production you would want to use the ``RedisTrackerStore`` to restore
after restarting the application. It's straightforward to define a
custom ``TrackerStore`` subclass for the persistence tool of your choice.


Serialisation
-------------

Rather than pickling the final state of the tracker object, 
Rasa creates a ``dialogue`` object to serialise.
A dialogue is a full record of the previous ``N`` dialogue turns. 
To return to the current state of the conversation,
we iterate over the turns and log the events in each.

We use the ``jsonpickle`` library to serialise these Dialogues.
Here's a simple example of a dialogue as it would be stored in the TrackerStore:

.. literalinclude:: ../data/test_dialogues/greet.json
    :language: json

