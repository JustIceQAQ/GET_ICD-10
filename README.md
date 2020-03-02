# GET_ICD-10
used python to get ICD-10 data

# Data Sources
https://icd.who.int/browse10/2019/en

I used browser DevTools check this webside requests/response.

I find ICD-10 data API response
![ICD-10 DevTools](https://imgur.com/dAnnA9K.png)

So...I used pandas read_json to read json data, like this:
Used this level "label" Value to search next level data.

```python
# Level 0
YEAR  = '2019'
ICDTree1 = pd.read_json(f'https://icd.who.int/browse10/{YEAR}/en/JsonGetRootConcepts?useHtml=false')

```

And then Level 1, Level 2, Level 3 like this:
```python

# Level 1
ICDTree1['label'] = ICDTree1.apply(clean_label, axis=1)
ICDTree2list = []
for lv1 in tqdm(ICDTree1.index, ascii=True, desc='RUN LV1'):
    ICDTree2raw = pd.read_json(
        'https://icd.who.int/browse10/{}/en/JsonGetChildrenConcepts?ConceptId={}&useHtml=false&showAdoptedChildren=true'.format(
            YEAR, ICDTree1.at[lv1, "ID"]))
    ICDTree2raw['LV1_ID'] = ICDTree1.at[lv1, 'ID']
    ICDTree2raw['LV1_label'] = ICDTree1.at[lv1, 'label']
    ICDTree2list.extend(ICDTree2raw.to_dict('r'))
ICDTree2 = pd.DataFrame(ICDTree2list)

ICDTree2['label'] = ICDTree2.apply(clean_label, axis=1)
# Level 2
ICDTree3list = []
for lv1 in tqdm(ICDTree2.index, ascii=True, desc='RUN LV2'):
    ICDTree3raw = pd.read_json(
        'https://icd.who.int/browse10/{}/en/JsonGetChildrenConcepts?ConceptId={}&useHtml=false&showAdoptedChildren=true'.format(
            YEAR,ICDTree2.at[lv1, "ID"]))
    ICDTree3raw['LV1_ID'] = ICDTree2.at[lv1, 'LV1_ID']
    ICDTree3raw['LV1_label'] = ICDTree2.at[lv1, 'LV1_label']
    ICDTree3raw['LV2_ID'] = ICDTree2.at[lv1, 'ID']
    ICDTree3raw['LV2_label'] = ICDTree2.at[lv1, 'label']
    ICDTree3list.extend(ICDTree3raw.to_dict('r'))
ICDTree3 = pd.DataFrame(ICDTree3list)
ICDTree3['label'] = ICDTree3.apply(clean_label, axis=1)


# Level 3
ICDTree4list = []
for lv1 in tqdm(ICDTree3.index, ascii=True, desc='RUN LV3'):
    ICDTree4raw = pd.read_json(
        'https://icd.who.int/browse10/{}/en/JsonGetChildrenConcepts?ConceptId={}&useHtml=false&showAdoptedChildren=true'.format(
            YEAR,ICDTree3.at[lv1, "ID"]))
    ICDTree4raw['LV1_ID'] = ICDTree3.at[lv1, 'LV1_ID']
    ICDTree4raw['LV1_label'] = ICDTree3.at[lv1, 'LV1_label']
    ICDTree4raw['LV2_ID'] = ICDTree3.at[lv1, 'LV2_ID']
    ICDTree4raw['LV2_label'] = ICDTree3.at[lv1, 'LV2_label']
    ICDTree4raw['LV3_ID'] = ICDTree3.at[lv1, 'ID']
    ICDTree4raw['LV3_label'] = ICDTree3.at[lv1, 'label']
    ICDTree4list.extend(ICDTree4raw.to_dict('r'))
ICDTree4 = pd.DataFrame(ICDTree4list)
ICDTree4['label'] = ICDTree4.apply(clean_label, axis=1)
ICDTree4.rename({'ID': 'LV4_ID', 'label': 'LV4_label'}, axis=1, inplace=True)

ICDTree4[['LV1_ID', 'LV1_label', 'LV2_ID', 'LV2_label', 'LV3_ID', 'LV3_label', 'LV4_ID', 'LV4_label']]
```
