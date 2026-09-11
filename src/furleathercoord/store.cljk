(ns furleathercoord.store
  "SSoT for the ISCO-08 8155 fur and leather preparing machine operators
  plant scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607121000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a plant scheduling/logistics coordination robot performs
  crew scheduling, production-run/inventory/progress-record logging
  and processing-chemicals/raw-hide supply-order coordination for a
  fur and leather preparing machine-operator crew under this
  advisor/governor pair, which never dispatches hardware itself, never
  operates fur/leather preparing (tanning/dressing) machinery itself,
  and never finalizes a processing-operation-execution decision or a
  chemical-safety-clearance decision, and never overrides a plant
  safety officer's judgment — those remain the plant safety officer's
  exclusive judgment). Modeled closely on cloud-itonami-isco-8122's
  platingcoord.store.

  Domain:

    operator — a registered fur and leather preparing machine operator
               crew member (:operator-id, :name)
    plant    — a registered fur/leather preparing plant/line
               {:plant-id :name :max-supply-cost number}.
               `:max-supply-cost` is an informational registered
               ceiling used only to decide whether a
               `:coordinate-supply-order` proposal escalates to human
               sign-off (the governor never blocks a within-threshold
               order outright; it only decides commit vs. escalate).
    record   — a committed operating record (a logged production-run/
               inventory/progress entry, a scheduled crew/shift
               operation, a flagged safety concern, or a coordinated
               processing-chemicals/raw-hide supply order) — written
               ONLY via commit-record!. This actor coordinates plant
               scheduling/logistics ONLY — a `record` is a
               coordination artifact, never a processing-operation-
               execution act, never a chemical-safety-clearance
               decision, and never a plant safety officer's-judgment
               override.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (operator [s operator-id])
  (plant [s plant-id])
  (records-of [s operator-id])
  (ledger [s])
  (register-operator! [s operator])
  (register-plant! [s plant])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (operator [_ operator-id] (get-in @a [:operators operator-id]))
  (plant [_ plant-id] (get-in @a [:plants plant-id]))
  (records-of [_ operator-id] (filter #(= operator-id (:operator-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-operator! [s o]
    (swap! a assoc-in [:operators (:operator-id o)] o) s)
  (register-plant! [s p]
    (swap! a assoc-in [:plants (:plant-id p)] p) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:operators {} :plants {} :records [] :ledger []}
                                    seed)))))
