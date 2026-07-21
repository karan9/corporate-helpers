# RSU Capital Gains & FSI Calculator

A standalone, browser-based working-paper tool for calculating Indian capital gains on foreign-company RSUs using E\*TRADE Gain & Loss workbooks.

The application imports one or more E\*TRADE `.xlsx` files, filters transactions for the selected Indian financial year, converts vesting cost and sale proceeds into INR using separately entered SBI TT buying rates, and prepares Schedule CG and Schedule FSI working papers.

> **Important:** This is a tax working-paper and estimation tool. It does not file an income-tax return and is not a substitute for professional tax advice.

---

## 1. Main features

- Runs entirely inside the web browser.
- No installation, server, login, or database is required.
- Supports multiple E\*TRADE Gain & Loss `.xlsx` workbooks.
- Includes a built-in spreadsheet reader, so an internet connection is not required after downloading the HTML file.
- Filters sales by Indian financial year, from 1 April through 31 March.
- Removes duplicate sale lots imported from repeated or overlapping files.
- Automatically reads the E\*TRADE USD vesting cost for each sold lot.
- Uses separate SBI TT buying rates for:
  - vesting or acquisition cost; and
  - sale proceeds.
- Groups transactions sharing the same specified TT-rate date, so each rate is entered only once.
- Calculates INR sale consideration, INR cost of acquisition, transfer expenses, and capital gain or loss.
- Automatically classifies transactions as short term or long term using an editable holding-period threshold.
- Allows manual overrides for:
  - short-term or long-term classification;
  - vest-wise INR cost of acquisition; and
  - transfer expenses in INR.
- Prepares consolidated Schedule CG working figures.
- Prepares Schedule FSI working figures when there is positive foreign-source capital-gain income.
- Performs basic current-year short-term and long-term capital-loss set-off within the imported transactions.
- Estimates tax using editable STCG slab rate, LTCG rate, surcharge, and cess.
- Exports detailed transaction workings, Schedule CG, and Schedule FSI as CSV files.
- Saves and restores the complete project as JSON.
- Supports browser print or Save as PDF.
- Retains the working locally in browser storage for convenience.

---

## 2. Files required

The application is designed for E\*TRADE's expanded Gain & Loss workbook format, commonly labelled **G&L Expanded**.

Only modern Excel `.xlsx` files are supported. Legacy `.xls` files must first be opened in a spreadsheet application and saved as `.xlsx`.

The parser looks for fields such as:

- Record Type
- Symbol
- Plan Type
- Quantity
- Date Acquired
- Vest Date
- Date Sold
- Total Proceeds
- Ordinary Income Recognized
- Ordinary Income Recognized Per Share
- Adjusted Cost Basis
- Adjusted Cost Basis Per Share
- Adjusted Gain/Loss
- Vest Date FMV
- Grant Number
- Order Number

The parser skips summary rows and imports individual sale lots.

### Important limitation of a Gain & Loss workbook

An E\*TRADE Gain & Loss workbook generally contains sold lots. It may not contain every vested share that is still held or every share withheld for payroll taxes.

Therefore, the workbook can normally support the capital-gains calculation for sold shares, but it may not be sufficient to reconcile the employer's complete annual RSU perquisite total reported in Form 12BA.

---

## 3. How the USD cost is selected

For each imported RSU sale lot, the application determines the USD vesting or perquisite cost using the following priority:

1. **Ordinary Income Recognized Per Share** multiplied by quantity.
2. **Adjusted Cost Basis Per Share** multiplied by quantity.
3. Total **Adjusted Cost Basis**.
4. Total **Ordinary Income Recognized**.
5. **Vest Date FMV** multiplied by quantity.
6. Acquisition-cost fields as a final fallback.

The preferred calculation is:

```text
USD vesting cost
= Quantity sold × Ordinary Income Recognized Per Share
```

For RSUs, E\*TRADE often reports the taxable vesting value under `Ordinary Income Recognized Per Share`, and the same amount may also appear under `Adjusted Cost Basis Per Share`.

The application warns the user when:

- ordinary income per share and adjusted cost basis per share differ materially;
- the reconstructed total USD cost differs from E\*TRADE's reported adjusted cost basis; or
- required cost or proceeds fields are missing.

E\*TRADE may sometimes format `Vest Date FMV` as an Excel date serial. For this reason, the application prefers the ordinary-income and adjusted-cost-basis fields.

---

## 4. Vesting-cost conversion into INR

The application assumes, by default, that the employer recognised the RSU salary perquisite in the same month in which the shares vested.

Under that assumption:

```text
Vesting TT specified date
= Last day of the month immediately preceding the vesting month
```

For example:

| Vesting date | Assumed payroll recognition month | Vesting TT date |
|---|---|---|
| 1 July 2025 | July 2025 | 30 June 2025 |
| 1 October 2025 | October 2025 | 30 September 2025 |
| 2 January 2026 | January 2026 | 31 December 2025 |

The INR cost is calculated as:

```text
INR cost of acquisition
= USD vesting cost × SBI USD TT buying rate for the vesting TT date
```

Example:

```text
Quantity sold                         4 shares
Ordinary income per share             USD 335.90
USD vesting cost                      USD 1,343.60
SBI TT buying rate on 30 June         INR 85.50 per USD
INR cost of acquisition               INR 114,877.80
```

### When to use the INR cost override

Use the per-lot INR cost override when:

- payroll recognised the RSU perquisite in a month different from the vesting month;
- the employer provides an exact vest-wise INR perquisite value;
- Form 12BA or a payroll statement provides a more authoritative vest-wise value; or
- the E\*TRADE cost field does not represent the amount taken into account for salary-perquisite taxation.

Form 12BA normally provides an aggregate annual perquisite figure, not a vest-wise breakdown. It can be used as a reconciliation control, but the full annual total should not automatically be assigned to only the shares sold.

---

## 5. Sale-proceeds conversion into INR

For the sale side, the application uses the sale or trade date reported by E\*TRADE.

```text
Sale TT specified date
= Last day of the month immediately preceding the sale month
```

Examples:

| E\*TRADE sale date | Sale TT date |
|---|---|
| 1 July 2025 | 30 June 2025 |
| 29 August 2025 | 31 July 2025 |
| 3 October 2025 | 30 September 2025 |
| 15 January 2026 | 31 December 2025 |

The INR sale consideration is calculated as:

```text
INR sale consideration
= Gross USD Total Proceeds × SBI USD TT buying rate for the sale TT date
```

The relevant date is based on the trade date, not:

- settlement date;
- wire-transfer date;
- date money reached an Indian bank account; or
- actual conversion rate received from a bank.

---

## 6. Exchange rate to enter

Enter the **SBI USD Telegraphic Transfer Buying Rate**, commonly displayed as:

- TT Buying;
- USD/INR TT Buy; or
- Telegraphic Transfer Buying Rate.

Do not use a generic market rate or substitute it with:

- Google or XE rate;
- RBI reference rate;
- E\*TRADE conversion rate;
- actual remittance conversion rate;
- SBI TT selling rate;
- card rate; or
- bills buying rate.

The application does not automatically download or verify SBI rates. The user must enter and retain evidence of each rate used.

---

## 7. Capital-gain calculation

For each sale lot:

```text
Capital gain or loss in INR
= INR sale consideration
− INR cost of acquisition
− eligible transfer expenses in INR
```

The application keeps transfer expenses separate from gross proceeds and cost of acquisition.

A user can manually enter eligible expenses attributable to a lot, such as a transaction-related brokerage or sale expense, after independently confirming its tax treatment.

---

## 8. Short-term and long-term classification

The application compares the acquisition or vesting date with the sale date.

The default foreign-share long-term threshold is **24 months**, and a holding period exceeding the configured threshold is classified as long term.

The threshold is editable because the applicable treatment may depend on the asset, transfer date, and law relevant to the selected assessment year.

Each transaction can also be manually changed to:

- Short term;
- Long term; or
- Automatic classification.

The exact Schedule CG category available in the current ITR utility should be checked before transferring the totals.

---

## 9. Current-year loss set-off

Within the imported transactions, the application performs a basic capital-gain set-off working:

- short-term capital loss may be used against positive short-term capital gain;
- any remaining short-term capital loss may be used against positive long-term capital gain; and
- long-term capital loss is not used against short-term capital gain.

The output shows:

- taxable short-term capital gain;
- taxable long-term capital gain;
- short-term loss used against long-term gain; and
- unabsorbed short-term or long-term loss.

This does not determine whether a loss is legally eligible for carry-forward. Filing timelines, prior-year losses, return status, and other income or gains are outside the application.

---

## 10. Tax estimate

The application estimates tax only on the transactions imported into the current project.

Editable inputs include:

- marginal tax slab for short-term capital gain;
- long-term capital-gain rate;
- surcharge; and
- health and education cess.

The estimate follows this structure:

```text
STCG tax
= Taxable STCG × selected STCG rate

LTCG tax
= Taxable LTCG × selected LTCG rate

Base tax
= STCG tax + LTCG tax

Surcharge
= Base tax × selected surcharge rate

Cess
= (Base tax + surcharge) × selected cess rate
```

The application does not model:

- basic exemption adjustment;
- rebate;
- marginal relief;
- special surcharge caps;
- other salary, interest, business, or capital-gain income;
- deductions;
- advance-tax interest;
- self-assessment-tax interest; or
- tax already deducted or paid.

The tax figure should therefore be treated as an estimate for the uploaded RSU transactions, not the final return liability.

---

## 11. Schedule CG working paper

The Schedule CG section consolidates short-term and long-term foreign-share transactions into:

- full value of consideration;
- cost of acquisition;
- cost of improvement, shown as zero;
- expenditure on transfer;
- capital gain or loss;
- taxable amount after current-year set-off;
- unabsorbed loss; and
- estimated tax.

The application provides a suggested category description, but the final category must be matched with the exact item in the assessment-year-specific ITR utility.

The Schedule CG working can be exported as CSV.

---

## 12. Schedule FSI working paper

Schedule FSI output is generated only when:

- the residential-status setting indicates that FSI is applicable;
- all required sale and vesting rates are available, or an INR cost override has been entered; and
- the imported calculation results in positive foreign-source capital-gain income.

The working includes:

- source country;
- country code;
- foreign TIN or other permitted identifier;
- head of income;
- foreign-source income included in INR;
- foreign tax paid in INR;
- Indian tax attributable;
- relief working; and
- DTAA article.

Foreign tax paid and the DTAA article must be entered manually.

The tool does not independently determine:

- foreign-tax-credit eligibility;
- the correct DTAA article;
- Form 67 compliance;
- Schedule TR reporting; or
- documentary requirements for claiming relief.

Where no positive capital-gain income remains, the tool keeps the loss in the capital-gain working and does not generate a positive FSI income figure.

---

## 13. What the application does not cover

This focused version does not prepare or reconcile:

- Schedule Salary;
- Form 12BA perquisite computation;
- Schedule FA A2 or A3;
- dividends or interest;
- Schedule OS;
- Schedule TR;
- Form 67;
- advance-tax calculations;
- interest under the Income-tax Act;
- marginal relief;
- complete loss carry-forward eligibility;
- previous-year losses;
- broker cash balances;
- actual foreign-bank account reporting; or
- the complete income-tax return.

Schedule FA can follow a different reporting period from Schedule CG and should be prepared separately using the applicable return instructions.

---

## 14. How to use the HTML file

1. Download `rsu_capital_gains_fsi_calculator.html`.
2. Open the file in a current desktop browser.
3. Select the relevant Indian financial year.
4. Review the STCG, LTCG, surcharge, cess, and holding-period settings.
5. Upload one or more E\*TRADE Gain & Loss `.xlsx` files.
6. Review import warnings and duplicate-lot messages.
7. Enter the requested SBI TT buying rates for each sale-rate date.
8. Enter the requested SBI TT buying rates for each vesting-rate date.
9. Review each sale lot and, where necessary, enter:
   - an INR cost override;
   - an eligible transfer expense; or
   - a manual short-term or long-term classification.
10. Review the Schedule CG and loss-set-off working.
11. Complete the Schedule FSI information only where applicable.
12. Export the detailed working and schedules as CSV.
13. Save the project JSON for future editing.
14. Use Print / Save PDF for a readable working-paper copy.

---

## 15. Saving and restoring work

The application automatically attempts to retain project data in the browser's local storage.

It also supports:

- **Save project JSON**, which downloads the complete working; and
- **Load project JSON**, which restores a previously saved project.

The project JSON may contain financial and tax information. Store it securely.

Clearing browser data, using private browsing, changing devices, or browser restrictions may remove local storage. The JSON export is the safer backup method.

---

## 16. Privacy and data handling

The application is browser-only.

- Uploaded spreadsheets are read locally in the browser.
- The application does not intentionally upload files to a server.
- No account or sign-in is required.
- No analytics or tracking service is included in the HTML file.
- Imported files and calculations remain on the device unless the user exports or shares them.

Users should still follow normal security precautions when handling tax files, broker statements, and project JSON backups.

---

## 17. Browser compatibility

Use a current version of a modern desktop browser, such as:

- Google Chrome;
- Microsoft Edge;
- Mozilla Firefox; or
- Safari.

The browser must support:

- JavaScript;
- FileReader and ArrayBuffer APIs;
- DOMParser;
- Blob downloads;
- local storage; and
- browser printing.

Very old browsers and restricted corporate browser environments may not work correctly.

---

## 18. Troubleshooting

### Nothing happens after selecting a workbook

- Confirm that the file extension is `.xlsx`.
- Re-export the workbook from E\*TRADE or save it again as `.xlsx`.
- Re-download the HTML file and reopen it.
- Try a current desktop version of Chrome or Edge.
- Review the import-status message shown below the upload area.

### No transactions are displayed

- Confirm that the workbook is the expanded Gain & Loss report.
- Confirm that it contains individual `Sell` records rather than only a summary.
- Confirm that the sale dates fall within the selected Indian financial year.

### Cost is missing or looks incorrect

- Review the E\*TRADE cost source displayed for the lot.
- Confirm that Ordinary Income Recognized Per Share or Adjusted Cost Basis is present.
- Compare quantity multiplied by the per-share cost with the reported adjusted cost basis.
- Use an INR cost override when payroll provides the authoritative Indian value.

### A vesting TT date appears incorrect

The default assumes payroll recognition in the same month as vesting. Use an INR cost override where payroll recognised the perquisite in another month or provided an exact INR vest-wise value.

### Schedule FSI is empty

FSI is not generated when:

- residential status is set to FSI not applicable;
- required TT rates are incomplete; or
- the net foreign-source capital-gain income is zero or negative.

---

## 19. Record-keeping checklist

Retain, as applicable:

- E\*TRADE Gain & Loss workbook;
- E\*TRADE vesting or release confirmations;
- employer payslips;
- Form 16;
- Form 12BA;
- evidence of SBI TT buying rates used;
- sale confirmations or contract notes;
- expense evidence;
- exported transaction working CSV;
- Schedule CG and FSI working CSVs;
- project JSON backup; and
- the final ITR computation used for filing.

---

## 20. Free-use permission

This application and its original source code are **free to use**.

You may use, copy, modify, adapt, and redistribute the application for personal, educational, professional, or commercial purposes without paying a fee or requesting prior permission.

When redistributing a modified version:

- do not represent the tool as an official Income Tax Department, SBI, E\*TRADE, Morgan Stanley, or employer product;
- preserve applicable third-party copyright and licence notices; and
- make clear that tax calculations should be independently verified.

Attribution is appreciated but not required.

The application is provided **as is**, without warranties or guarantees of accuracy, fitness for a particular purpose, legal compliance, or continued compatibility with E\*TRADE exports or Indian tax-return utilities. The user remains responsible for reviewing all inputs, assumptions, calculations, and return disclosures.

### Third-party component

The HTML file embeds **JSZip 3.10.1** for reading `.xlsx` files. JSZip remains subject to its own MIT or GPLv3 licensing terms and copyright notices. Those notices should be retained when the application is redistributed.

---

## 21. Disclaimer

Tax law, return forms, reporting requirements, exchange-rate rules, and ITR utility fields may change. The application does not automatically verify the law applicable to a particular taxpayer, assessment year, residential status, employer plan, or transaction.

Before filing:

- verify the applicable Income-tax Act provisions and rules;
- verify the selected assessment-year ITR utility;
- reconcile the figures with Form 16, Form 12BA, AIS, broker records, and tax payments; and
- obtain advice from a qualified tax professional where the facts or treatment are uncertain.

The presence of a calculation in this tool does not establish that a deduction, expense, loss, foreign-tax credit, or reporting position is legally available.

---
