# Pipeline documentation

## Senario : Emails

### Expected input

We suppose that user has finished preprocessing of the original emial files. The input file of this pipeline should maintain a structure as followed:

| Message-ID |     Date |  From |    To | Subject | Mime-Version | Content-Type |    Content-Transfer-Encoding | X-From |            X-To |              X-cc | X-bcc | X-Folder |                                  X-Origin | Content |
| ---------: | -------: | ----: | ----: | ------: | -----------: | -----------: | ---------------------------: | -----: | --------------: | ----------------: | ----: | -------: | ----------------------------------------: | ------: |
|          0 | Normtime | email | Email |         |              |          1.0 | text/plain; charset=us-ascii |   7bit | Phillip K Allen | Christi L Nicolay |   NaN |      NaN | \Phillip_Allen_Dec2000\Notes Folders\Sent | Allen-P |

It should maintain āt least 3 features:

- **Date**: the date sending this email

- **From**: the sender of this email

- **Content**: the content of the email after tokenizing

### transformers

- | Parameter Name | Description                                                                                                                                          |
  | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
  | emailID        | the row number of this email in the original csv file                                                                                                |
  | Event          | the 'Date' column of the original csv file                                                                                                           |
  | Object         | the 'From' column of the original csv file                                                                                                           |
  | Activity       | String like '**activity topic : 1**' or ''**no activity** '. We get this string by applying DBSCAN on the 'Content' column of the original csv file  |
  | event logs     | a dataframe whose columns are ['emailID','Object','Event','Activity'], each row represent a qualified event log whose activity is not 'no activity'. |
  | OE             | a dataframe whose columns are ['emailID','Object','Event']                                                                                           |
  | EA             | a dataframe whose columns are ['emailID','Event', 'Activity']                                                                                        |

- all the transformers:

  | Transformers                  | Description                                                                                                                                                                              |
  | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | EventExtractionTransformer    | read the original emails csv file, and return a dataframe ['**emailID**', '**Event**']                                                                                                   |
  | ObjectExtractionTransformer   | read the original emails csv file, and return a dataframe ['**emailID**', '**Object**']                                                                                                  |
  | ActivityExtractionTransformer | read the original emails csv file, and return a dataframe ['**emailID**', '**Activity**']                                                                                                |
  | OEA_Combiner                  | read the output of **ObjectExtractionTransformer**, **EventExtractionTransformer** and **ActivityExtractionTransformer**, then combine 3 seperate dataframe into one **event logs** data |
  | OE_Combiner                   | read the output of **ObjectExtractionTransformer**, **EventExtractionTransformer**, then combine 2 seperate dataframe into one **OE** dataframe.                                         |
  | EA_Combiner                   | read the output of **EventExtractionTransformer**, **ActivityExtractionTransformer**, then combine 2 seperate dataframe into one **EA** dataframe.                                       |
  | OE_EA_Combiner                | read the output of **OE_Combiner**, **EA_Combiner**, then combine 2 seperate dataframe into one **event logs** dataframe.                                                                |

- **ObjectExtractionTransformer**: takes the input csv file of the emails, and return a list of extracted **objects**(sender in this case). The index of this list related to the number of emails in the csv file.

  ```python
  class ObjectExtractionTransformer(BaseEstimator, TransformerMixin):
      '''ObjectExtractionTransformer
      Extract objects from emails

      Parameters
      ----------
      data: pandas.DataFrame
          The data to be transformed

          Returns
          -------
          dataframe: ['emailID', 'object']
      '''
      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
          objects_list = X['From'].to_list()
          objects = pd.DataFrame(columns=['emailID', 'object'])
          for i in range(len(objects_list)):
              objects.loc[len(objects)] = [i, objects_list[i]]
          return objects
  ```

- **EventExtractionTransformer**: takes the input csv file of the emails, and return a list of extracted **events**(date sending email). The index of this list related to the number of emails in the csv file.

  ```python
  class EventExtractionTransformer(BaseEstimator, TransformerMixin):
      '''ActivityExtractionTransformer
      Extract activities from emails

      Parameters
      ----------
      data: pandas.DataFrame
          The data to be transformed

          Returns
          -------
          dataframe: ['emailID', 'event']'''

      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
          events_list = X['Date'].to_list()
          events = pd.DataFrame(columns=['emailID', 'event'])
          for i in range(len(events_list)):
              events.loc[len(events)] = [i, events_list[i]]
          return events
  ```

- **ActivityExtractionTransformer**: takes the input csv file of the emails, and return a list of extracted **activities**, here we use **DBSCAN** algorithms to calculate the clusters in thoes emails and then use the clustered topics as extracted activities. For users, they can replace by using **experts notation**.

  The activity list it returns will contain the tag like : activity topic: N, or no activity which means no activities extracted from this email.

  ```python

  class ActivityExtractionTransformer(BaseEstimator, TransformerMixin):
      '''Activity Extraction Transformer
      Extract activities from emails

      Parameters
      ----------
      data: pandas.DataFrame

          Returns
          -------
          dataframe: ['emailID', 'activity']
      '''

      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
              contents = X['Content']

              nlp = spacy.load('en_core_web_sm')
              sent_vecs = {}
              docs = []

              # for each email content i, get the vector representation of the content
              # and store it in sent_vecs
              for i in range(len(contents)):
                  doc = nlp(contents[i])
                  docs.append(doc)
                  sent_vecs[i] = doc.vector

              sentenses = list(sent_vecs.keys())
              vectors = list(sent_vecs.values())

              # cluster emails ito different topics based on the vector representation using DBSCAN
              x = np.array(vectors)

              n_classes = {}

              for i in tqdm(np.arange(0.001, 1, 0.002)):
                  dbscan = DBSCAN(eps=i, min_samples=2, metric='cosine').fit(x)
                  n_classes.update({i: len(pd.Series(dbscan.labels_).value_counts())})

              dbscan = DBSCAN(eps=0.08, min_samples=2, metric='cosine').fit(x)

              results = pd.DataFrame({'sentense': sentenses, 'cluster': dbscan.labels_})
              clusters = results.groupby('cluster')['sentense'].apply(list)
              num_clusters = len(clusters)

              activity_list = [-1]*len(contents)
              #activities = pd.DataFrame(columns=['emailID', 'activity'])

              for i in range(num_clusters -1): # cluster[-1] means emails without any activities
                  for j in clusters[i]:
                      if clusters[i] != -1:
                          activity_list[j] = 'activity topic ' + str(i)
              for j in clusters[-1]:
                  activity_list[j] = "no activity"

              activities = pd.DataFrame(columns=['emailID', 'activity'])
              for i in range(len(activity_list)):
                  activities.loc[len(activities)] = [i, activity_list[i]]
              return activities

  ```

- **OEA_Combiner**： This trasformer takes 3 input which are events, activities and objects, and then combine them together. The input events, activities and objects are the results from previous preprocessors, and they should have the same length.

  ```python
  class OEA_Combiner(BaseEstimator, TransformerMixin):
      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
          event_logs = pd.DataFrame(columns=['emailID', 'event', 'object', 'activity'])

          # get the event, object and activity of an email
          # events is the first 2 columns of X
          emailID = X[:,0]
          events = X[:,1]
          objects = X[:,3]
          activities = X[:,5]

          for e in emailID:
              if activities[e] == 'no activity':
                  continue
              else:
                  event_logs.loc[len(event_logs)] = [e, events[e], objects[e], activities[e]]
          return event_logs
  ```

- **OE_Combiner**：read the output of **ObjectExtractionTransformer**, **EventExtractionTransformer**, then combine 2 seperate dataframe into one **OE** dataframe.

  ```Python
  class OE_Combiner(BaseEstimator, TransformerMixin):
      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
          # split X into event and object
          emailID = X[:,0]
          events = X[:,1]
          objects = X[:,3]

          # create a dataframe
          events_objects = pd.DataFrame(columns=['emailID', 'event', 'object'])

          for e in emailID:
              events_objects.loc[len(events_objects)] = [e, events[e], objects[e]]
          return events_objects
  ```

- **EA_Combiner**: read the output of **EventExtractionTransformer**, **ActivityExtractionTransformer**, then combine 2 seperate dataframe into one **EA** dataframe.

  ```python
  class EA_Combiner(BaseEstimator, TransformerMixin):
      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          return self

      def transform(self, X, y=None):
          # split X into event and activity
          emailID = X[:,0]
          events = X[:,1]
          activities = X[:,3]

          event_activities = pd.DataFrame(columns=['emailID', 'event', 'activity'])

          for e in emailID:
              event_activities.loc[len(event_activities)] = [e, events[e], activities[e]]
          return event_activities
  ```

- **OE_EA_Combiner**:read the output of **OE_Combiner**, **EA_Combiner**, then combine 2 seperate dataframe into one **event logs** dataframe.

  ```python
  class OE_EA_Combiner(BaseEstimator, TransformerMixin):
      def __init__(self, data):
          self.data = data

      def fit(self, X, y=None):
          self.data = X
          return self

      def transform(self, X, y=None):
          event_logs = pd.DataFrame(columns=['emailID', 'event', 'object', 'activity'])

          # get the event, object and activity of an email
          events_objects = X[:, :3]
          events_activities = X[:, 3:]

          num = 0
          for i in range(len(X)):
              # insert the event log into the last row of the dataframe
              if events_activities[i][2] != 'no activity':
                  event_logs.loc[num] = [i, events_objects[i][1], events_objects[i][2], events_activities[i][2]]
                  num += 1

          return event_logs
  ```

### Pipelines

- **Pipeline 1**: generate objects, events and activities seperately and then combine these three together.

![pipeline_1](https://p.ipic.vip/4kcqa5.png)

​

```python
steps_union = FeatureUnion([
    ('event_extraction', EventExtractionTransformer(df)),
    ('object_extraction', ObjectExtractionTransformer(df)),
    ('activity_extraction', ActivityExtractionTransformer(df))
])

pipe = Pipeline([
    ('parallel', steps_union),
    ('combination', CombinationTransformer(df))
])
```

![截屏2023-03-21 00.40.03](https://p.ipic.vip/gzeae5.png)

- **Pipeline 2**: first generate

  ![](https://p.ipic.vip/2jekzz.png)

```python
# pipeline 2
Pip2_ParallelStep_1 = FeatureUnion([
    ('event_extraction', EventExtractionTransformer(emails)),
    ('object_extraction', ObjectExtractionTransformer(emails))
])

Pip2_ParallelStep_2 = FeatureUnion([
    ('event_extraction', EventExtractionTransformer(emails)),
    ('activity_extraction', ActivityExtractionTransformer(emails))
])

# get [events] and [objects] then combine them
Pip2_SubPipeline_1 = Pipeline([
    ('Extract objects and events seperately', Pip2_ParallelStep_1),
    ('combine objects and events', EO_Combiner(emails))
])

# get [events] and [activities] then combine them
Pip2_SubPipeline_2 = Pipeline([
    ('Extract events and activities seperately', Pip2_ParallelStep_2),
    ('combine events and activities', EA_Combiner(emails))
])

Pip2_ParallelStep_3 = FeatureUnion([
    ('combine objects and events', Pip2_SubPipeline_1),
    ('combine events and activities', Pip2_SubPipeline_2)
])

pipeline_2 = Pipeline([
    ('get (events, objects) and (events,activities)', Pip2_ParallelStep_3),
    ('combine_object_event_activities', EO_EA_Combiner(emails))
])

pipeline_2.fit_transform(emails)
```

### How to use the tool

- **Step 1** User need to upload the path of the file from which they want to extract event logs. This file must in required format.

  ![](https://p.ipic.vip/xhnmrq.jpg)

- **Step 2** Uer select the senario they want to use. Most of the current algorithms about extracting evnet logs focus on certain case, and different cases may have different need of constructure of pipelines. By choosing different sennarios, user can then select pipelines within that case senario.

  ![select senario](https://p.ipic.vip/ns8jgm.jpg)

- **Step 3** User select pipelines within selected case senario.

  ![select pipeline](https://p.ipic.vip/9wyfwg.jpg)

- **Step 4** Run the selected pipeline on the given file, and show the result.

  ![run certain pipeline](https://p.ipic.vip/a72h2g.jpg)

  ![show the result of the pipeline](https://p.ipic.vip/8h23nd.jpg)
