(ns civil-labour.governor
  "CivilLabourGovernor — the independent safety/traceability layer for the
  ISCO-08 9312 independent civil-labour-crew actor. The Site Advisor
  proposes actions (labor task, inspection); it has no notion of work-order
  provenance or excavation safety, so this MUST be a separate system able to
  *reject* a proposal and fall back to HOLD — the itonami-actor pattern
  (independent Governor gates a proposing actor) applied to this occupation.

  Charter (mirrors ADR-2607011000 robotics premise + ADR-2607012000
  cloud-itonami-isco): the actor never dispatches a robot action or writes an
  operating record the governor refuses. A labor task near an excavation
  ALWAYS requires human sign-off, even when every hard invariant passes.

  HARD invariants for :civil-labour/propose:
    1. Work-order provenance — a labor task or inspection must reference a
       registered work order on a registered site.
    2. No-actuation          — the proposal must not directly mutate a
       labor-task or inspection record outside the record-labor-task!/
       record-inspection! path (effect must be :propose, never a raw store
       write).
    3. Excavation safety     — a labor task of kind :near-excavation on a
       site with `has-excavation? true` always requires :high or higher
       safety-class, which forces human sign-off; it is never
       auto-approved regardless of confidence.
  SOFT:
    4. Confidence floor → escalate."
  (:require [civil-labour.store :as store]))

(def confidence-floor 0.6)
(def safety-classes [:none :low :medium :high :safety-critical])

(defn- safety-rank [safety-class]
  (let [idx (.indexOf safety-classes safety-class)]
    (if (neg? idx) 0 idx)))

(defn- hard-violations [{:keys [order-fn site-fn]} proposal]
  (let [{:keys [kind order-id safety-class effect]} proposal
        found-order (order-fn order-id)
        found-site  (when found-order (site-fn (:site-id found-order)))]
    (cond-> []
      (nil? found-order)
      (conj {:rule :no-work-order :detail (str "未登録 work-order " order-id)})

      (not= :propose effect)
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（直接書込禁止）"})

      (and (= kind :near-excavation) found-site (:has-excavation? found-site)
           (< (safety-rank (or safety-class :none)) (safety-rank :high)))
      (conj {:rule :excavation-safety
             :detail ":near-excavation task は :high 以上の safety-class が必須"}))))

(defn assess
  "Assess a proposal against `env` (a map with `:order-fn`/`:site-fn`
  lookups, decoupled from any concrete Store so this stays pure). Returns
  `{:decision :proceed|:hold|:human-approval :violations [...] :confidence n}`."
  [env proposal]
  (let [violations (hard-violations env proposal)
        safety-class (or (:safety-class proposal) :none)
        confidence (or (:confidence proposal) 0.0)]
    (cond
      (seq violations)
      {:decision :hold :violations violations :confidence confidence}

      (>= (safety-rank safety-class) (safety-rank :high))
      {:decision :human-approval :violations [] :confidence confidence}

      (< confidence confidence-floor)
      {:decision :human-approval :violations [] :confidence confidence
       :reason :low-confidence}

      :else
      {:decision :proceed :violations [] :confidence confidence})))

(defn env-for-store
  "Build the decoupled env map `assess` needs from a concrete
  `civil-labour.store/Store` implementation."
  [store]
  {:order-fn #(store/work-order store %)
   :site-fn  #(store/site store %)})
