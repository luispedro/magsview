# Adapting MAGs View to a different MAG catalog

The app ships with a baked-in set of MAG metadata (the table on the Genomes page) and
per-genome detail JSON files (used on each genome detail page). To swap in another
catalog, update both pieces of data so they stay in sync.

## 1) Prepare the MAG metadata CSV

The genome table is powered by a CSV string embedded in `src/Data/Blobs/MAGs.elm`.
Create a CSV file with the following header row and columns:

```
id,samples_id,taxonomy,completeness,contamination,genome_size,#16s_rrna,#5s_rrna,#23s_rrna,#trna,nr_contigs,nr_genes,is_representative,binning_tool,assembly_method,comment,file_size,file_sha256
```

Notes:
- `samples_id` accepts comma-separated sample identifiers (e.g., `S001,S002`).
- `is_representative` should be `True` or `False` (capitalized) to match the Elm decoder.
- Numeric columns must be parseable as numbers; leave `comment` empty if not applicable.

Once the CSV looks correct, replace the string literal assigned to `magsBlob` in
`src/Data/Blobs/MAGs.elm` with the new CSV content (keep the surrounding
`String.trim """ ... """` structure intact). The `LoadData.loadData` helper will parse
it at startup.

## 2) Add per-genome JSON files

Each genome detail page fetches `/genome-data/<MAG_ID>.json` from the `static/genome-data`
folder. For every `id` in the CSV, create a matching JSON file named
`<MAG_ID>.json` with this structure:

```json
{
  "16S": [
    { "Seq": "<16S_sequence>", "OTU": "<otu_metadata>" }
  ],
  "ARGs": [
    {
      "Sequence": "<amino_acid_sequence>",
      "ArgName": "<ARG_name>",
      "ARO": "<ARO_identifier>",
      "Cut_Off": "<e.g., Perfect/Strict>",
      "Identity": <float>,
      "Coverage": <float>,
      "InResfinder": <true|false>,
      "Drug Class": "<drug_class_label>"
    }
  ]
}
```

You can include multiple entries in the `16S` and `ARGs` arrays. Make sure the JSON is
compact or pretty-printed—Elm’s decoder only depends on the field names and types.

## 3) Verify locally

After updating the CSV and JSON files:

1. Run `npx elm-land server` to rebuild the site.
2. Visit `/genomes` and click into a genome to ensure the metadata and ARG/16S sections
   render without parse errors.

Keep the CSV and JSON files in sync: every MAG listed in the CSV should have a matching
JSON file, and vice versa.
