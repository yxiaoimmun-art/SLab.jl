Act as a systems engineer specialized in laboratory automation, cyber-physical systems, and scheduling optimization.

Do NOT explain biology concepts.

Transform the following protocol into a systems-engineering-style lab automation operation diagram.

The objective is NOT a biological workflow chart.

The objective is:
- automation orchestration
- resource scheduling
- operation synchronization
- TCMB-constrained execution planning
- laboratory MES-style operation modeling

Represent the workflow as:

- modular task nodes
- instrument resource lanes
- operators
- state transitions
- data flow edges
- synchronization barriers
- QC checkpoints
- temporal constraints
- parallel execution regions
- blocking/non-blocking tasks
- reusable instrument resources

Layout requirements:

1. Instruments must be arranged horizontally as persistent resource lanes.
2. Operations should flow across the instrument lanes.
3. Repeated instrument usage must reuse the same lane.
4. Highlight synchronization and QC explicitly.
5. Show waiting states and timing dependencies.
6. Emphasize sample-state transitions.
7. Distinguish human operations from automated operations.
8. Include resource contention and bottlenecks.

For every operation node include:
- task name
- input state
- output state
- duration
- required resource
- blocking/non-blocking behavior
- synchronization condition
- QC trigger
- upstream/downstream dependencies

Use TCMB (Time Constraints by Mutual Boundaries) to model:
- minimum wait time
- maximum delay tolerance
- synchronization windows
- coupled timing constraints

Output format:
1. Experiment modular decomposition
2. Resource mapping table
3. TCMB constraint matrix
4. Operation DAG
5. Instrument-lane operation diagram
6. Automation scheduling logic
7. Critical path analysis
8. Bottleneck analysis

Use terminology from:
- discrete event systems
- manufacturing scheduling
- orchestration engines
- industrial automation
- cyber-physical systems

The protocol is:
#### 3.4 Measurement & Calculation
- **Function:** `measure_absorbance(wavelength)`
  - Input: developed plate, 562 nm
  - Output: absorbance readings (OD562)
  - QC Point: Blank OD <0.1

- **Function:** `calculate_protein_concentration(absorbance_data, standard_curve_data)`
  - Input: sample OD values, BSA standard curve (linear regression)
  - Output: protein concentration (μg/μL) per sample
  - QC Point: R² >0.99 for standard curve

- **Function:** `normalize_samples(raw_lysates, target_concentration)`
  - Input: variable concentration lysates, equal concentration target
  - Output: normalized protein samples
  - QC Point: Concentration variance <10%

**Data Flow:** BSA Stock → Standard Series → Loaded Plate → Incubated Plate → Absorbance Values → Concentration Data → Normalized Samples

---

### MODULE 4: PROTEIN DENATURATION
**Responsibility:** Prepare proteins for electrophoretic separation

#### 4.1 Loading Buffer Preparation
- **Function:** `prepare_loading_buffer(concentration_factor, reducing_agent, agent_concentration)`
  - Input: 4× loading buffer, DTT or β-mercaptoethanol, 100mM DTT or 5% β-ME
  - Output: complete denaturing buffer
  - QC Point: Reducing agent freshness

#### 4.2 Sample Denaturation
- **Function:** `mix_sample_with_buffer(normalized_protein, loading_buffer, mixing_ratio)`
  - Input: normalized lysate, 4× buffer, 3:1 ratio (sample:buffer)
  - Output: protein-buffer mixture
  - QC Point: Thorough vortexing

- **Function:** `heat_denature(protein_mixture, temperature, duration)`
  - Input: protein mix, 95°C, 5 min
  - Output: denatured protein complexes
  - QC Point: Heat block calibration

- **Function:** `cool_on_ice(denatured_sample)`
  - Input: hot protein mix, ice bath
  - Output: cooled denatured sample
  - QC Point: Rapid cooling (<2 min)

- **Function:** `centrifuge_briefly(sample_tube)`
  - Input: cooled tube, brief spin
  - Output: condensed sample at tube bottom
  - QC Point: No condensation on tube walls

**Data Flow:** Normalized Protein → Mixed with Buffer → Heated Mixture → Cooled Denatured Protein → Condensed Ready-to-Load Sample

---

### MODULE 5: SDS-PAGE ELECTROPHORESIS
**Responsibility:** Separate proteins by molecular weight

#### 5.1 Gel System Setup
- **Function:** `assemble_gel_system(gel_specification, electrophoresis_apparatus)`
  - Input: 4-12% Bis-Tris precast gel, Mini-PROTEAN Tetra system
  - Output: assembled gel cassette in tank
  - QC Point: Gel integrity (no cracks/leaks)

- **Function:** `add_running_buffer(electrophoresis_tank, buffer_type)`
  - Input: tank, 1× MOPS or MES running buffer
  - Output: filled electrophoresis chamber
  - QC Point: Buffer level above gel top

#### 5.2 Sample Loading
- **Function:** `load_samples(denatured_proteins, protein_load_amount, molecular_weight_marker)`
  - Input: denatured samples, 20-30 μg per well, prestained marker 3-5 μL
  - Output: loaded gel with samples + ladder
  - QC Point: No sample overflow between wells

#### 5.3 Electrophoretic Run
- **Function:** `run_electrophoresis(applied_voltage, run_duration)`
  - Input: loaded gel, 120-150V, 50-60 min
  - Output: separated protein bands in gel
  - QC Point: Dye front migration position

**Data Flow:** Precast Gel → Filled Tank → Loaded Samples → Running Gel → Separated Protein Bands

---

### MODULE 6: PROTEIN TRANSFER
**Responsibility:** Transfer proteins from gel to membrane for antibody access

#### 6.1 Membrane Activation
- **Function:** `activate_membrane(membrane_type, activation_method)`
  - Input: nitrocellulose membrane, methanol or water activation
  - Output: wetted, activated membrane
  - QC Point: Uniform wetting (no dry spots)

#### 6.2 Transfer Assembly
- **Function:** `assemble_transfer_sandwich(separated_gel, activated_membrane, filter_papers, sponges)`
  - Input: gel post-electrophoresis, NC membrane, transfer filter paper, sponges
  - Output: layered transfer cassette (correct orientation critical)
  - QC Point: No air bubbles between layers

#### 6.3 Electrotransfer
- **Function:** `transfer_proteins(applied_current, transfer_temp, transfer_duration)`
  - Input: assembled sandwich, 250-300mA, 4°C, 60-90 min
  - Output: proteins immobilized on membrane
  - QC Point: Current stability, temperature maintenance

**Data Flow:** Separated Gel + Activated Membrane → Assembled Sandwich → Transferred Membrane with Immobilized Proteins

---

### MODULE 7: ANTIBODY INCUBATION & WASHING
**Responsibility:** Specific detection of target proteins via immunoreaction

#### 7.1 Membrane Blocking
- **Function:** `block_membrane(blocking_agent, blocking_temp, blocking_time)`
  - Input: transferred membrane, 5% skim milk in TBS-T, RT, 1h
  - Output: blocked membrane (non-specific sites occupied)
  - QC Point: Complete coverage of membrane surface

#### 7.2 Primary Antibody Incubation
- **Function:** `incubate_primary_antibody(target_protein, antibody_source, dilution_factor, incubation_temp, incubation_time)`
  - Input: blocked membrane, anti-ACSL4/GPX4/4-HNE, manufacturer-recommended dilution, 4°C, overnight
  - Output: membrane with primary antibody bound to target
  - QC Point: Antibody specificity validation
