# AI Audit

## Prompt 1

### Message
"Можеш проаналізувати предметну область разом з усіма use-cases а також sequence-діаграм та порекомендувати одну з форм опису бізнес-процесу однієї точної формою (або Activity або BPMN або State Machine?)".

### Result
Activity-diagram was selected to describe one business process.<br>
Activity-diagram fits well, because UC contains:
- Sequential operations
- Conditions and checks
- Alternative scenarious
- Cycles
- User interaction with the system

### Why not BPMN or State Machine
- BPMN would be too complex notation for this specific domen.<br>
- State machine describes lifecycle and state of objects, but UC mostly describe operations - not change of state

## Prompt 2
### Message
"На основі проаналізованої предметної області згенеруй activity діаграму. Вимоги та критерії прописані у spec.md"

### Result
Generated `activity-diagram.md`
