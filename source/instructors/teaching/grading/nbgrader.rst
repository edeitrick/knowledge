.. meta::
   :description: Auto-Grade Jupyter notebook assignments using nbgrader.
   
.. _notebooks:


Auto-Grade with nbgrader
========================
Codio supports `Jupyter notebook <https://jupyter.org/>`_ auto-grading functionality through `nbgrader <http://nbgrader.readthedocs.io/en/stable/index.html>`_. Assignments are created with `Jupyter notebook <https://jupyter.org/>`_ and when the assignment is published to a course, the release version is created for the student. If the assignment is updated and republished, it overwrites all tests and read-only cells with the new version and the release version for the students is updated.

When a student submits the assignment by marking the assignment as complete, the assignment is automatically graded. However, manual grading is also possible if desired. 

User configurations for nbgrader can be stored in a **nbgrader_config.py** or in **.codio-jupyter** file. A .codio-jupyter file must be present in a project to let Codio know that nbgrader should be used to grade Jupyter assessments.  

.. Important:: If using **nbgrader_config.py** for your configurations, the **.codio-jupyter** is still required but can be empty/blank

.. Note:: If both files are used the settings in the **nbgrader_config.py** take precedence. This file is not visible to students in their assignments 

.. Note:: Notebook files are only supported if in the root (`/home/codio/workspace` or `~/workspace`) folder

.. Warning:: :ref:`Pair Programming <group-work>` should not be used for Jupyter Notebook

Configuration
-------------
Use the following configuration information when setting up nbgrader in a **.codio-jupyter** file. If using **nbgrader_config.py**, see :ref:`example <nb-conf-example>` below.

- **Extend Timeout period** - To extend the time required for completion (to 90 seconds in this example), you can add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     ExecutePreprocessor.timeout: 90
 

- **Lock all cells** - To lock all cells (Default: False), add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     LockCells.lock_all_cells: True



- **Lock all grade cells** - To lock all grade cells (Default: True) where grade cells are locked (non-deletable), add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     LockCells.lock_grade_cells: True


- **Lock all read-only cells** - To lock all grade cells (Default: True) where read only cells are locked (non-deletable and non-editable), add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     LockCells.lock_readonly_cells: True


- **Lock all solution cells** - To lock all solution cells (Default: True) where solution cells are locked (non-deletable and non-editable), add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     LockCells.lock_solution_cells: True


- **Execute preprocessor on timeout** - If execution of a cell times out, interrupt the kernel and continue executing other cells rather than throwing an error and stopping by adding the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
     ExecutePreprocessor.interrupt_on_timeout: True


- **Run custom grading with Jupyter** - To avoid execution of autograder with nbgrader and allow Codio script autograder to be executed, add the following to the **.codio-jupyter** file. When this is set, Jupyter files do not display as assessments in Codio and are not submitted through nbrader after the assignment is marked as completed (no assessments and points are set in the assignment).

.. code:: yaml

  codio:
    grader: false


- **ClearSolutions.code_stub** - Add the following to the **.codio-jupyter** file:

.. code:: yaml

  nbgrader:
      ClearSolutions.code_stub:
          R: |
              # BEGIN YOUR CODE
              # END YOUR CODE
          python: |
              # YOUR CODE HERE
              raise NotImplementedError()
          ruby: |
              # BEGIN YOUR CODE
              raise NotImplementedError.new()
              #END YOUR CODE
  
.. _postgrading:

- **Postgrader**       

You can add a post-grading hook to Jupyter to alter the result html for the student. You can do this to remove and/or replace text from the notebook file that the students will see in their feedback.

.. code:: yaml

  codio:
    postGrader: .guides/secure/postgrader.py

To enable this, create a file **postgrader.py** in .guides/secure folder. This file needs to be executable.
Running ```chmod +x .guides/secure/postgrader.py``` will make this file executable.

Example postgrader.py file
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    #!/usr/bin/env python3
    import sys
    import re
    START = '<span class="c1">### BEGIN HIDDEN TESTS</span>'
    END = '<span class="c1">### END HIDDEN TESTS</span>'
    html_path = sys.argv[1].rstrip()
    with open(html_path, 'r') as content_file:
        content = content_file.read()

    def replaceTextBetween(originalText, delimeterA, delimterB, replacementText):
        index_from = 0
        index_to = len(originalText)
        if delimeterA in originalText:
            index_from = originalText.index(delimeterA)

        if delimterB in originalText:
            index_to = originalText.index(delimterB) + len(delimterB)

        return originalText[0:index_from] + originalText[index_to:]

    while START in content:
        content = replaceTextBetween(content, START, END, '')
    with open(html_path, 'w+') as stream:
        stream.write(content)


In this example anything between the ### BEGIN HIDDEN TESTS and ### END HIDDEN TESTS in the **.ipynb** file will not be shown to the students 
  
If using the **nbgrader_config.py**, see example below

.. _nb-conf-example:

Example nbgrader_config.py
--------------------------

.. code:: python

    c = get_config()
    c.ClearHiddenTests.begin_test_delimeter = "BEGIN HIDDEN TESTS"
    c.ClearHiddenTests.end_test_delimeter = "END HIDDEN TESTS"
    c.LockCells.lock_all_cells = True
    c.LockCells.lock_grade_cells = True
    c.LockCells.lock_readonly_cells = True
    c.LockCells.lock_solution_cells = True
    c.ExecutePreprocessor.interrupt_on_timeout = True
    c.ExecutePreprocessor.timeout = 20
    c.ClearSolutions.code_stub = {
    "R": "# your R code here\n# end of R code\n",
    "python": "# your python code here\n# end of python code\n",
    "ruby": "# your ruby code here            \n# end of ruby code"
    }
    

