# Mirroring test
This test cases requires an extra manual steps to fully test. It ensures that any mirrored resources that inherit annotations from the original resource are not managed by the agent and won't be accidentally deleted.

After deployment is created, copy the yaml and update only the name. Redeploy it in the same namespace and check that it is not deleted by the agent.