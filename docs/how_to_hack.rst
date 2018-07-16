How to Hack
===========

There are several places where you can incorporate your own ideas and needs into IEPY.
Here you'll see how to modify different parts of the iepy core.

Altering how the corpus is created
----------------------------------

On the `preprocess <preprocess.html#how-to-customize>`_ section was already mentioned that you can customize how the corpus is created.


Using your own classifier
-------------------------

You can change the definition of the *extraction classifier* that is used when running
iepy in *active learning* mode.

As the simplest example of doing this, check the following example.
First, define your own custom classifier, like this:

.. code-block:: python

    from sklearn.linear_model import SGDClassifier
    from sklearn.pipeline import make_pipeline
    from sklearn.feature_extraction.text import CountVectorizer


    class MyOwnRelationClassifier:
        def __init__(self, **config):
            vectorizer = CountVectorizer(
                preprocessor=lambda evidence: evidence.segment.text)
            classifier = SGDClassifier()
            self.pipeline = make_pipeline(vectorizer, classifier)

        def fit(self, X, y):
            self.pipeline.fit(X, y)
            return self

        def predict(self, X):
            return self.pipeline.predict(X)

        def decision_function(self, X):
            return self.pipeline.decision_function(X)


and later, in iepy_runner.py of your IEPY instance, in the **ActiveLearningCore** creation,
provide it as a configuration parameter like this


.. code-block:: python

    iextractor = ActiveLearningCore(
        relation, labeled_evidences,
        tradeoff=tuning_mode,
        extractor_config={},
        extractor=MyOwnRelationClassifier
    )


Implementing your own features
------------------------------

Your classifier can use features that are already built within iepy or you can create your
own. You can even use a rule (as defined in the :doc:`rules core <rules_tutorial>`) as feature.

Start by creating a new file in your instance, you can call it whatever you want, but for this
example lets call it ``custom_features.py``. There you'll define your features:

.. code-block:: python

    # custom_features.py
    from featureforge.feature import output_schema

    @output_schema(int, lambda x: x >= 0)
    def tokens_count(evidence):
        return len(evidence.segment.tokens)


.. note::

    Your features can use some of the `Feature Forge's <http://feature-forge.readthedocs.org/en/latest/>`__
    capabilities.

Once you've defined your feature you can use it in the classifier by adding it to the configuration
file. You should have one on your instance with all the default values, it's called ``extractor_config.json``.

There you'll find 2 sets of features where you can add it: dense or sparse. Depending on the values returned
by your feature you'll choose one over the other.

To include it, you have to add a line with a python path to your feature function. If you're not familiarized with
the format you should follow this pattern:

::

    {project_name}.{features_file}.{feature_function}

In our example, our instance is called ``born_date``, so in the config this would be:

.. code-block:: json

    "dense_features": [
        ...
        "born_date.custom_features.tokens_count",
        ...
    ],

Remember that if you want to use that configuration file you have to use the option ``--extractor-config``


Using rules as features
-----------------------

In the same way, and without doing any change to the rule, you can
add it as feature by declaring it in your config like this:

Suppose your instance is called ``born_date`` and your rule is called ``born_date_in_parenthesis``,
then you'll do:


.. code-block:: json

    "dense_features": [
        ...
        "born_date.rules.born_date_in_parenthesis",
        ...
    ],

This will run your rule as a feature that returns 0 if it didn't match and 1 if it matched.

Using all rules as one feature
..............................

Suppose you have a bunch of rules defined in your rules file and instead of using each rule as a
different feature you want to use a single feature that runs all the rules to test if the evidence
matches. You can write a custom feature that does so. Let's look an example snippet:

.. code-block:: python

    # custom_features.py
    import refo

    from iepy.extraction.rules import compile_rule, generate_tokens_to_match, load_rules

    rules = load_rules()


    def rules_match(evidence):
        tokens_to_match = generate_tokens_to_match(evidence)

        for rule in rules:
            regex = compile_rule(rule, evidence.relation)

            if refo.match(regex, tokens_to_match):
                if rule.answer:  # positive rule
                    return 1
                else:  # negative rule
                    return -1
        # no rule matched
        return 0


This will define a feature called ``rules_match`` that tries every rule for an evidence
until a match occurs, and returns one of three different values, depending on the type
of match.

To use this you have to add this single feature to your config like this:

.. code-block:: json

    "dense_features": [
        ...
        "born_date.custom_features.rules_match",
        ...
    ],



Documents Metadata
------------------

While building your application, you might want to store some extra information about your documents.
To avoid loading this data every time when predicting, we've separated the place to put this 
information into another model called **IEDocumentMetadata** that is accessible through the **metadata** attribute.

IEDocumentMetadata has 3 fields:

    * title: for storing document's title
    * url: to save the source url if the document came from a web page
    * itmes: a dictionary that you can use to store anything you want.

By default, the **csv importer** uses the document's metadata to save the filepath of the csv file on the *items* field.
