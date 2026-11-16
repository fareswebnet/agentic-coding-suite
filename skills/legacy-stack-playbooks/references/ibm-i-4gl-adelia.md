# IBM i generated applications: Adelia and similar 4GL or CASE tools

Status: unvalidated against a live system. Adelia facts come from vendor pages (Hardis: the platform generates RPG ILE, Java, C, and HTML5/CSS3 depending on the module); anything else below is a checklist of questions to confirm with the tool's owners. Read `ibm-i.md` first; it still applies to the generated side.

## The key difference with hand-written RPG

The business logic lives in the tool's repository (models, screens, actions, and the generator's own conventions), and the RPG or Java on the machine is generated output. Inventorying only the generated source misses the real definition and reads like noise.

## Inventory sources

- The tool's repository or export: programs, screens, data model, shared functions, versions; the way to export it in a readable, scriptable form is the first thing to establish. If the tool can only be browsed in its IDE, build the extractor and test it before freezing the registry.
- The generated artifacts on the IBM i (and on other platforms if the tool generates several), with the generator version that produced each one.
- Hand-edited generated code: compare generated source against a fresh generation; any difference is custom logic hidden outside the model.
- Runtime and generator libraries, shared service programs, and configuration data the generated programs call.
- Usage: the object usage data and job histories of `ibm-i.md`, mapped back from generated object names to repository items.
- Quality tooling the tool offers (for Adelia Studio a quality module with a Sonar plugin is announced by the vendor) can speed up the inventory; check its output against your own extractor.

## Hidden behavior to turn into rules or questions

- Defaults the generator applies: rounding, null or empty handling, date formats, field truncation, error and message handling, locking, and commitment behavior. They are not in the model; they are in the generator and its runtime, and they change with the generator version.
- Behavior that differs between two generation targets (for instance RPG ILE and Java) of the same model.
- Screen flow logic generated from model properties, including implicit validations and function key behavior.
- Embedded native code (RPG, SQL, CL calls) written inside the tool.

## Strategy hints

- Decide whether the source of the rules is the repository (readable, but needs generator semantics) or the running behavior (always true, harder to read); use both, and let the oracle run on the real application decide.
- If the tool also generates a target you could keep (for example Java or web output), evaluate it as a rehost or transform option in `migration-strategy`, and test its output with the same oracles as hand-written code.
- Other generated-code tools on IBM i (for instance Synon or 2E, LANSA) follow the same pattern; confirm their export and generator conventions before relying on this list.

## Oracle design

Run the generated application with the same captured inputs as for RPG, and compare database files, queues, spooled output, and messages. Re-run after a regeneration with a new generator version to see whether behavior moves.
