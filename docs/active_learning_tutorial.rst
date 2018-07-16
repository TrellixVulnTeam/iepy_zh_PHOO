Running the active learning core
================================

The active learning core works by trying to predict the relations using information provided by the user.
This means you'll have to label some of the examples and based on those, the core will infer the rest.
The core will also give you to label the more important examples (those which best helps
to figure out the other cases).

To start using it you'll need to define a relation, run the core, label some evidence and re-run the core loop.
You can also label evidences and re-run the core as much as you like to have a better performance.

Creating a relation
-------------------

To create a relation, first `open up the web server <tutorial.html#open-the-web-interface>`__ if you haven't already, and use a
web browser to navigate on `http://127.0.0.1:8000 <http://127.0.0.1:8000>`_.
There you'll find instructions on how to create a relation.

Running the core
----------------

After creating a relation, you can start the core to look for instances of that relation.

You can run this core in two modes: **High precision** or **high recall**.
`Precision and recall <http://en.wikipedia.org/wiki/Precision_and_recall>`_ can be traded with one another up to a certain point.  I.e. it is possible to trade some
recall to get better precision and vice versa.

To visualize better this trade off, lets see an example:
A precision of 99% means that 1 of every 100 predicted relations will be wrong and the rest will be correct.
A recall of 30% means that only 30 out of 100 existent relations will be detected by the algorithm and the rest
will be wrongly discarded as "no relation present".

Run the active learning core by doing:

.. code-block:: bash

    python bin/iepy_runner.py <relation_name> <output>

And add ``--tune-for=high-prec`` or ``--tune-for=high-recall`` before the relation name to switch
between modes. The default is **high precision**.

This will run until it needs you to label some of the evidences. At this point, what you
need to do is go to the web interface that you ran on the previous step, and there you
can label some evidences.

When you consider that is enough, on the prompt that the iepy runner presented you,
continue the execution by typing **run**.

That will cycle again and repeat the process.

Run the active learning core in the command line and ask it to **STOP**.
It'll save a csv with the automatic classifications for all evidences in the database.

Also, note that you can only predict a relation for a text that has been inserted into the database.
The csv output file has the primary key of an object in the database that represents the evidence that 
was classified as "relation present" or "relation not present". An evidence object in the database is a
rich-in-information object containing the entities and circumstances surrounding the prediction that 
is too complex to put in a single csv file.

In order to access the entities and other details you'll need to write a script 
to talk with the database (see iepy/data/models.py).


Fine tuning
-----------

If you want to modify the internal behavior, you can change the settings file. On your instance
folder you'll fine a file called ``extractor_config.json``. There you've all the configuration
for the internal classifier, such as:

Classifier
..........

This sets the classifier algorithm to be used, you can choose from:

    * sgd: `Stochastic Gradient Descent <http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.SGDClassifier.html>`_
    * knn: `Nearest Neighbors <http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsClassifier.html#sklearn.neighbors.KNeighborsClassifier>`_
    * svc `(default)`: `C-Support Vector Classification <http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html>`_
    * randomforest: `Random Forest <http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html>`_
    * adaboost: `AdaBoost <http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.AdaBoostClassifier.html>`_

Features
........

Features to be used in the classifier, you can use a subset of:

    * number_of_tokens
    * symbols_in_between
    * in_same_sentence
    * verbs_count
    * verbs_count_in_between
    * total_number_of_entities
    * other_entities_in_between
    * entity_distance
    * entity_order
    * bag_of_wordpos_bigrams_in_between
    * bag_of_wordpos_in_between
    * bag_of_word_bigrams_in_between
    * bag_of_pos_in_between
    * bag_of_words_in_between
    * bag_of_wordpos_bigrams
    * bag_of_wordpos
    * bag_of_word_bigrams
    * bag_of_pos
    * bag_of_words

These can be added as `sparse` adding them into the
`sparse_features` section or added as `dense` into the `dense_features`.

The features in the sparse section will go through a stage of linear dimension reduction
and the dense features, by default, will be used with a non-linear classifier.


Viewing predictions on the web user interface
---------------------------------------------

If you prefer to review the predictions using the web interface is possible to run the
active learning core in a way that stores the results on the database and they are accesible
through the web.

To do so, you'll have to run the core like this:

.. code-block:: bash

    python bin/iepy_runner.py --db-store <relation_name> 

We do not have an specialized interface to review predictions but you can still view them
by using the :doc:`interface to create a reference corpus <corpus_labeling>`.

This way, you'll get labels as a new **judge** called iepy-run and a date.

.. image:: labels_by_iepy.png


Saving predictor for later use
------------------------------

Since training could be a slow process, you might want to save your trained predictor and
re-use it several times without the need to train again.

You can save it this by doing:

.. code-block:: bash

    python bin/iepy_runner.py --store-extractor=myextractor.pickle <relation_name> <output>

And re use it like this:

.. code-block:: bash

    python bin/iepy_runner.py --trained-extractor=myextractor.pickle <relation_name> <output>
