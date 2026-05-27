import { useEffect, useState } from "react";
import {
  Alert,
  Avatar,
  Box,
  Button,
  Chip,
  CircularProgress,
  Paper,
  Stack,
  TextField,
  Typography,
} from "@mui/material";
import EditIcon from "@mui/icons-material/Edit";
import { getMe, updateMyProfile } from "../api/userApi";
import type { User } from "../api/userApi";
import { getTags } from "../api/tagApi";
import type { Tag } from "../api/tagApi";
import type { ProblemDetails } from "../api/apiFetch";

export default function MyPage(): JSX.Element {
  const [me, setMe] = useState<User | undefined>(undefined);
  const [tagOptions, setTagOptions] = useState<Tag[]>([]);

  const [errorMessage, setErrorMessage] = useState("");
  const [isEditing, setIsEditing] = useState(false);
  const [isSaving, setIsSaving] = useState(false);

  const [department, setDepartment] = useState("");
  const [bio, setBio] = useState("");
  const [selectedTagIds, setSelectedTagIds] = useState<number[]>([]);

  useEffect((): void => {
    async function fetchData(): Promise<void> {
      try {
        const [fetchedMe, fetchedTags] = await Promise.all([
          getMe(),
          getTags(),
        ]);

        setMe(fetchedMe);
        setTagOptions(fetchedTags);

        setDepartment(fetchedMe.department ?? "");
        setBio(fetchedMe.bio ?? "");
        setSelectedTagIds(fetchedMe.tags?.map((tag) => tag.id) ?? []);
      } catch (error) {
        const problem = error as ProblemDetails;
        setErrorMessage(problem.detail ?? "データの取得に失敗しました。");
      }
    }

    void fetchData();
  }, []);

  function startEdit(): void {
    if (!me) return;

    setDepartment(me.department ?? "");
    setBio(me.bio ?? "");
    setSelectedTagIds(me.tags?.map((tag) => tag.id) ?? []);
    setErrorMessage("");
    setIsEditing(true);
  }

  function cancelEdit(): void {
    if (!me) return;

    setDepartment(me.department ?? "");
    setBio(me.bio ?? "");
    setSelectedTagIds(me.tags?.map((tag) => tag.id) ?? []);
    setErrorMessage("");
    setIsEditing(false);
  }

  function toggleTag(tagId: number): void {
    if (selectedTagIds.includes(tagId)) {
      setSelectedTagIds(selectedTagIds.filter((id) => id !== tagId));
    } else {
      setSelectedTagIds([...selectedTagIds, tagId]);
    }
  }

  async function saveProfile(): Promise<void> {
    try {
      setIsSaving(true);

      const updatedMe = await updateMyProfile({
        department,
        bio,
        tag: selectedTagIds,
      });

      setMe(updatedMe);
      setDepartment(updatedMe.department ?? "");
      setBio(updatedMe.bio ?? "");
      setSelectedTagIds(updatedMe.tags?.map((tag) => tag.id) ?? []);

      setIsEditing(false);
      setErrorMessage("");
    } catch (error) {
      const problem = error as ProblemDetails;
      setErrorMessage(problem.detail ?? "プロフィールの更新に失敗しました。");
    } finally {
      setIsSaving(false);
    }
  }

  if (!me) {
    return (
      <Box sx={{ p: 3 }}>
        <CircularProgress />
      </Box>
    );
  }

  return (
    <Box sx={{ p: 3 }}>
      {errorMessage && (
        <Alert severity="error" sx={{ mb: 2 }}>
          {errorMessage}
        </Alert>
      )}

      <Paper
        elevation={3}
        sx={{
          p: 4,
          mb: 4,
        }}
      >
        <Stack
          direction="row"
          spacing={5}
          alignItems="center"
        >
          <Box>
            <Avatar
              src={me.profileIconUrl ?? undefined}
              sx={{
                width: 180,
                height: 180,
                fontSize: 48,
                bgcolor: "primary.main",
              }}
            >
              {me.name.charAt(0)}
            </Avatar>

            {!isEditing ? (
              <Button
                startIcon={<EditIcon />}
                onClick={startEdit}
                sx={{ mt: 3 }}
              >
                編集
              </Button>
            ) : (
              <Stack direction="row" spacing={1} sx={{ mt: 3 }}>
                <Button
                  variant="contained"
                  onClick={saveProfile}
                  disabled={isSaving}
                >
                  保存
                </Button>

                <Button
                  variant="outlined"
                  onClick={cancelEdit}
                  disabled={isSaving}
                >
                  キャンセル
                </Button>
              </Stack>
            )}
          </Box>

          <Box sx={{ flex: 1 }}>
            <Typography variant="h4" sx={{ mb: 1 }}>
              {me.name}
            </Typography>

            {isEditing ? (
              <TextField
                label="部署"
                value={department}
                onChange={(e) => setDepartment(e.target.value)}
                size="small"
                sx={{ mb: 2, width: 240 }}
              />
            ) : (
              <Typography variant="h6" sx={{ mb: 2 }}>
                {me.department || "部署未設定"}
              </Typography>
            )}

            <Typography sx={{ mb: 3 }}>
              {me.email}
            </Typography>

            {isEditing ? (
              <TextField
                label="自己紹介"
                value={bio}
                onChange={(e) => setBio(e.target.value)}
                multiline
                minRows={3}
                fullWidth
                sx={{ mb: 3 }}
              />
            ) : (
              <Typography sx={{ mb: 3 }}>
                {me.bio || "自己紹介未設定"}
              </Typography>
            )}

            {isEditing ? (
              <Stack
                direction="row"
                spacing={1}
                flexWrap="wrap"
                useFlexGap
              >
                {tagOptions.map((tag) => (
                  <Chip
                    key={tag.id}
                    label={tag.name}
                    clickable
                    color={
                      selectedTagIds.includes(tag.id)
                        ? "primary"
                        : "default"
                    }
                    onClick={() => toggleTag(tag.id)}
                  />
                ))}
              </Stack>
            ) : (
              <Stack
                direction="row"
                spacing={1}
                flexWrap="wrap"
                useFlexGap
              >
                {me.tags.length > 0 ? (
                  me.tags.map((tag) => (
                    <Chip key={tag.id} label={tag.name} />
                  ))
                ) : (
                  <Typography color="text.secondary">
                    タグ未設定
                  </Typography>
                )}
              </Stack>
            )}
          </Box>
        </Stack>
      </Paper>

      <Box>
        <Typography variant="h6" sx={{ mb: 1 }}>
          参加したイベント
        </Typography>

        {/* ここに参加イベント一覧を置く */}
        <Typography color="text.secondary">
          demo2 詳細
        </Typography>
      </Box>
    </Box>
  );
}
