# LLM RAG Development workflow

## Intro:

- LLMs can generate assembly but struggle with RV32 due to its niche nature. So when observed with a structured prompt providing context on how the program should be, it was able to generate effectively:


```bash

input: 
Generate a RISC-V RV32 load store assembly program. In the assembly format like this. ##include<rvp.h>
 
RVP_TEXT 
<your instructions will go here>
 
RVP_DATA 
<data region will go here>
 
inside RVP_TEXT when test is done you can jump to pass routine which is defined in the rvp.h kernel, so just insert "j pass" instructions at the end of the test.
 
 
 
 
Output:
 
##include <rvp.h>
 
RVP_TEXT
    li t0, 10
    li t1, 20
    la t2, 0x1000
    sw t0, 0(t2)
    lw t3, 0(t2)
    addi t4, t3, 5
    j pass
 
RVP_DATA
   .word 0x1000
```


## Parsing:

- Since RV32 resources are highly structured, extracting relevant information efficiently is crucial. 

- There are open source OCR projects that can be utilized; one such project is OlmOCR [https://github.com/allenai/olmocr]. It can extract images, tables, charts and other such data from an input document using object recognition. 

- Another open source project is Marker[https://github.com/VikParuchuri/marker]. It converts PDFs and other similar documents into MarkDown, which is the best format for an LLM to understand structural representations.


## Embedding: 

- There are several embedding models that excel at different tasks. In order to pick one for our use case, MTEB ranks embeddings on retrieval, similarity, ranking, etc. We need to pick a model based on our use case—probably retrieval-focused.
- Since our primary use case is retrieval, we should focus on models that rank well in retrieval-specific MTEB benchmarks


## Storing: 

- Storing retrieved context in a vector database must be done locally and not in cloud storage.

- ChromaDB is an open sourced vector database that allows for local storing and is completely free of charge. If scaling up becomes an issue, it could be resolved by utilizing distributed database storing techniques.


## Retrieving and generation:

- Once stored, the data can be retrieved from the database in different methods. Different search metrics could also be used, either cosine similarity, Dot product similarity or other such ones. 

- Once retrieved, it has to be fed to the model. An approach I am planning on looking into is dynamic model selection, since not everyone might have know the best models for each use case, so having a system that dynamically selects the apt model based on the task given would be useful.

- Considering scaling aspect, if there are multiple requests at once, 

## Verifying output:

- Once the generation is done, the output (if it is a RV32 test code) could be verified by executing in the specsim simulator. Different metrics can be used to verify the output:

	1. Unit Tests: Run the generated assembly against predefined test cases and check if expected outputs match actual outputs.
	2. Instruction Count: Count the number of instructions generated and ensure it aligns with expected efficiency.
	3. Perturbation Tests: Modify input constraints slightly and check if the model generates consistent and correct outputs.
	4. Error Rate: Measure the percentage of invalid or incorrect assembly outputs and so on

- I have forked multiple repositories which have already implemented chat interfaces, I'll take some time playing around with each of them until I find one that I think would work.

- We could have a system which can compile code on the user interface (similar to openAI); where we could have an option for the user to run the code in the UI once it gets generated. That could be done by integrating a specsim terminal in the UI. Once .S is generated, it can be dumped into a path and after running the logs could be displayed on the interface.



