# Structuring a Dataset for Agentic Evaluation

## References

- [Vertex AI: Evaluation Dataset Documentation](https://cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-dataset)
- [Vertex AI Blog: Introducing Agent Evaluation](https://cloud.google.com/blog/products/ai-machine-learning/introducing-agent-evaluation-in-vertex-ai-gen-ai-evaluation-service)
- [Vertex AI: Trajectory Evaluation](https://cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-agents#trajectory_evaluation)

> To assess an agent using Vertex AI Gen AI evaluation service, you start preparing an evaluation dataset. This dataset should ideally contain the following elements:
>
> - **User prompt:** This represents the input that the user provides to the agent.
> - **Reference trajectory:** This is the expected sequence of actions that the agent should take to provide the correct response.
> - **Generated trajectory:** This is the actual sequence of actions that the agent took to generate a response to the user prompt.
> - **Response:** This is the generated response, given the agent's sequence of actions.

---

## Example: Reference and Predicted Trajectories

Below is an example of how to structure the `reference_trajectory` and `predicted_trajectory` for evaluation:

```python
reference_trajectory = [
    # example 1
    [
        {
            "tool_name": "set_device_info",
            "tool_input": {
                "device_id": "device_2",
                "updates": {
                    "status": "OFF"
                }
            }
        }
    ],
    # example 2
    [
        {
            "tool_name": "get_user_preferences",
            "tool_input": {
                "user_id": "user_y"
            }
        },
        {
            "tool_name": "set_temperature",
            "tool_input": {
                "location": "Living Room",
                "temperature": 23
            }
        },
    ]
]

predicted_trajectory = [
    # example 1
    [
        {
            "tool_name": "set_device_info",
            "tool_input": {
                "device_id": "device_3",
                "updates": {
                    "status": "OFF"
                }
            }
        }
    ],
    # example 2
    [
        {
            "tool_name": "get_user_preferences",
            "tool_input": {
                "user_id": "user_z"
            }
        },
        {
            "tool_name": "set_temperature",
            "tool_input": {
                "location": "Living Room",
                "temperature": 23
            }
        },
    ]
]

import pandas as pd

eval_dataset = pd.DataFrame({
    "predicted_trajectory": predicted_trajectory,
    "reference_trajectory": reference_trajectory,
})