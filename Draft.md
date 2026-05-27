import { useState } from "react";
import {
  Box,
  Button,
  Stack,
  TextField,
  Typography,
  Chip,
} from "@mui/material";
import { User, UpdateProfileRequest } from "../api/userApi";

type Props = {
  user: User;
  onSubmit: (request: UpdateProfileRequest) => Promise<void>;
  onCancel: () => void;
};

export default function ProfileForm({
  user,
  onSubmit,
  onCancel,
}: Props): JSX.Element {
  const [department, setDepartment] = useState(user.department ?? "");
  const [bio, setBio] = useState(user.bio ?? "");
  const [selectedTagIds, setSelectedTagIds] = useState<number[]>(
    user.tags?.map((tag) => tag.id) ?? []
  );

  const tagOptions = [
    { id: 3, name: "tag3" },
    { id: 4, name: "tag4" },
  ];

  function handleTagClick(tagId: number): void {
    if (selectedTagIds.includes(tagId)) {
      setSelectedTagIds(selectedTagIds.filter((id) => id !== tagId));
    } else {
      setSelectedTagIds([...selectedTagIds, tagId]);
    }
  }

  async function handleSave(): Promise<void> {
    await onSubmit({
      department,
      bio,
      tag: selectedTagIds,
    });
  }

  return (
    <Box sx={{ mt: 3 }}>
      <Stack spacing={2}>
        <TextField
          label="部署"
          value={department}
          onChange={(e) => setDepartment(e.target.value)}
          fullWidth
        />

        <TextField
          label="自己紹介"
          value={bio}
          onChange={(e) => setBio(e.target.value)}
          fullWidth
          multiline
          minRows={3}
        />

        <Box>
          <Typography sx={{ mb: 1 }}>タグ</Typography>

          <Stack direction="row" spacing={1}>
            {tagOptions.map((tag) => (
              <Chip
                key={tag.id}
                label={tag.name}
                clickable
                color={selectedTagIds.includes(tag.id) ? "primary" : "default"}
                onClick={() => handleTagClick(tag.id)}
              />
            ))}
          </Stack>
        </Box>

        <Stack direction="row" spacing={2} justifyContent="flex-end">
          <Button variant="outlined" onClick={onCancel}>
            キャンセル
          </Button>

          <Button variant="contained" onClick={handleSave}>
            保存
          </Button>
        </Stack>
      </Stack>
    </Box>
  );
}
