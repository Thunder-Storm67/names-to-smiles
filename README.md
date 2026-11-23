# names-to-smiles
A small command-line tool that converts a list of chemical compound names into SMILES strings and saves the results to a CSV file.  It reads names from a plain text file, looks each one up using free online chemistry services, and writes the SMILES (plus a few useful properties) to a CSV you can open in Excel, pandas, RDKit, or any other tool.

## Features
 
- Batch conversion from a simple `.txt` file (one compound name per line)
- Uses **PubChem** as the primary source, with **NCI CACTUS** as an automatic fallback
- Returns isomeric SMILES, connectivity SMILES, molecular formula, molecular weight, IUPAC name, and PubChem CID
- Names that can't be resolved are kept in the output with `status = not_found`, so nothing silently disappears
- Skips blank lines, `#` comments, and duplicate names
- Respects PubChem's rate limits and retries on temporary errors
- Writes results row by row, so an interrupted run keeps its partial output
## Requirements
 
- Python 3.9 or newer
- [`requests`](https://pypi.org/project/requests/)
- An internet connection
## Installation
 
```bash
git clone https://github.com/<your-username>/names-to-smiles.git
cd names-to-smiles
pip install requests
```
 
## Usage
 
1. Create a text file with one compound name per line:
```text
# compounds.txt
aspirin
caffeine
ibuprofen
2-acetoxybenzoic acid
```
 
2. Run the script:
```bash
python names_to_smiles.py compounds.txt
```
 
   This creates `compounds_smiles.csv` next to the input file.
 
### Options
 
| Option | Description | Default |
| --- | --- | --- |
| `input` | Text file with one compound name per line | required |
| `-o`, `--output` | Path of the output CSV file | `<input>_smiles.csv` |
| `--delay` | Seconds to wait between requests | `0.25` |
 
Example with options:
 
```bash
python names_to_smiles.py compounds.txt -o results.csv --delay 0.3
```
 
## Output
 
The CSV contains one row per compound name:
 
| Column | Description |
| --- | --- |
| `name` | The name exactly as it appeared in your input file |
| `status` | `found` or `not_found` |
| `source` | Which service returned the result (`PubChem` or `NCI CACTUS`) |
| `cid` | PubChem compound ID (PubChem results only) |
| `smiles` | Isomeric SMILES (includes stereochemistry where available) |
| `connectivity_smiles` | SMILES without stereochemistry (PubChem results only) |
| `formula` | Molecular formula (PubChem results only) |
| `molecular_weight` | Molecular weight (PubChem results only) |
| `iupac_name` | IUPAC name (PubChem results only) |
 
Example (illustrative):
 
```csv
name,status,source,cid,smiles,connectivity_smiles,formula,molecular_weight,iupac_name
aspirin,found,PubChem,2244,CC(=O)OC1=CC=CC=C1C(=O)O,CC(=O)OC1=CC=CC=C1C(=O)O,C9H8O4,180.16,2-acetyloxybenzoic acid
caffeine,found,PubChem,2519,CN1C=NC2=C1C(=O)N(C(=O)N2C)C,CN1C=NC2=C1C(=O)N(C(=O)N2C)C,C8H10N4O2,194.19,"1,3,7-trimethylpurine-2,6-dione"
not-a-real-compound,not_found,,,,,,,
```
 
## How it works
 
1. Each name is URL-encoded and sent to the PubChem PUG REST API.
2. If PubChem has no match, the name is sent to the NCI CACTUS chemical identifier resolver.
3. The result (or a `not_found` marker) is written to the CSV.
## Limitations
 
- **Ambiguous names:** a common name can match more than one compound. PubChem returns its top match, so spot-check results for ambiguous names.
- **Fallback data:** NCI CACTUS results only include SMILES, so the other columns are empty for those rows.
- **Large batches:** the script makes requests one at a time to stay within rate limits. For very large lists (thousands of names), consider adding a local cache or using PubChem's batch endpoints.
- **Validation:** SMILES are returned as provided by the services and are not validated locally. Tools like [RDKit](https://www.rdkit.org/) can be used for validation and canonicalisation.
## Data sources and acknowledgements
 
- [PubChem](https://pubchem.ncbi.nlm.nih.gov/) PUG REST API, provided by the U.S. National Library of Medicine. See the [PubChem usage policies](https://pubchem.ncbi.nlm.nih.gov/docs/programmatic-access).
- [NCI/CADD Chemical Identifier Resolver (CACTUS)](https://cactus.nci.nih.gov/chemical/structure), provided by the NCI Computer-Aided Drug Design Group.
Please follow the terms of use of these services, especially when running large batches.
 
## Contributing
 
Issues and pull requests are welcome.
  
