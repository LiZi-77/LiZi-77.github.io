# Event Extraction

Event extraction is an important task of information retrieval in natural language processing(NLP). The main goal of event extractionn task is to detect event in text, and identify the arugument of this event.

Event extraction techniques can be used in several domains:

- for goverments: they can detect the occurence of social events and prepare in advance
- for componies: they can use event extraction techniques to find business process patterns. Based on that, they can optimize their workflow.
- for the markets, they can extract events from history and predict the risk or the price change

For event extraction tasks, it still have some challengings, and many researchers have worked to overcome them:

- the data source are diverse. Some data sources are structured, like the database of an organization or porject. However, there still remains a lot of potential events from unnstructured sources, such as posts on Twitter, emails, an d some videos. And due to the explosion of internet, the trend that unstructured or multi-media data sources contain more information is being boosted.
- Event extraction task related to other NLP tasks such as **entity recognition**, **semantic role labeling** and so on. The findings in these related tasks will also influence the way to perform the event extraction tasks.

There have been a lot of researches focused on the event extraction tasks, the techniques of event extraction can be basicly devided into 2 categories:

1. closed-domain event extraction
   - Mainly focued on limited areas
   - limited event types and predefined event schema and event type and event arguments.
   - The process is first detect event trigger words, and then use the detected word to find the corresponding event type and its arguments
1. open-domian event extraction
   - aims to detect events in more general cases
   - first detecct events, and then cluster similiar events to the same topic.

Each of them has its own subtasks:

1. closed-domain:
   - trigger detection
   - event/trigger type identification
   - event argumment detection
   - argument role identification
2. open-domain:
   - event detection
     - story segmentation
     - first story detection
   - event clustering
     - topic detection
     - topic tracking
     - story link detection

There have been seperate researches on these subtasks, such as \cite{}.

However, no matter what kind of event extraction,closed-domain or open-domain, it envolves a combination of steps to perform sub-tasks. And different combinations of techniques used for these sub-tasks will influence the performence of the whole event extraction tasks. Different use senarios may have different preference of combinations.

We can use pipeline structure to combine the steps involved in the implementation of an event extraction model. And we also want to provide a tool which can help users to easily change the techniques used for subtasks or single steps of the whole implementation. That's the goal of our thesis project.

### Unsupervised learning

To perform a event extraction task, several kinds of techniques have been proposted, such as pattern matching, machine learning, deep learning and reinforcement learning.

The machine learning based techniques can be devided into supervised learning and unsupervised llearnning. For supervised approaches, the coverage of this data is limited, mainly for domian-specific applications and acquiring more labled data can be expensive.\cite{unsupervised}. However, unsupervised approaches are usually used to extract large numbers of untyped events.

paper \cite{unsupervised} proposed a pipeline for extraction annd cclustering complex events from news articles.
